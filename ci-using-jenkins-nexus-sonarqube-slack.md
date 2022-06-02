# CI using Jenkins, Nexus, SonarQube and Slack.


## Jenkins Setup

```
#! /bin/bash

sudo apt update; sudo apt upgrade -y
sudo apt install  -y ca-certificates maven git wget unzip openjdk-11-jdk
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt install -y jenkins

```

## Nexus Setup


Install OpenJDK 8

```
sudo apt install -y openjdk-8-jdk
```

Download Nexus

```
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

Extract Nexus

```
sudo tar -xvf latest-unix.tar.gz
```

Move Nexus & Sonatype-work to /opt

```
sudo mv nexus sonatype-word /opt/
```

Create a new user called nexus

```
sudo adduser nexus
```

Give permission to nexus user

```
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
```

Open nexus.rc file and edit or replace the following line

```
sudo vim /opt/nexus/bin/nexus.rc
```

chage the existing line to ```run_as_user="nexus"```

Modify memory settings

change first three lines with following lines

```
-Xms512m
-Xmx512m
-XX:MaxDirectMemorySize=512m
```

Configure Nexus to run as a service

```sudo vim /etc/systemd/system/nexus.service```

Copy the below content

```
[Unit]
Description=Nexus Service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
User=nexus
Group=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort
[Install]
WantedBy=multi-user.target

```

Now start Nexus

```
sudo systemctl enable nexus
sudo systemctl start nexus
sudo systemctl status nexus
```

For Nexus logs

```
tail -f /opt/sonatype-work/nexus3/log/nexus.log
```


Once Nexus is successfully installed, you can access it in the browser by

```URL: http://ip-address:port-number(8081)```







```
#! /bin/bash

sudo yum update -y
sudo yum install -y wget java-11-openjdk
mkdir -p /opt/nexus/   
mkdir -p /tmp/nexus/                           
cd /tmp/nexus
NEXUSURL="https://download.sonatype.com/nexus/3/latest-unix.tar.gz"
wget $NEXUSURL -O nexus.tar.gz
EXTOUT=`tar xzvf nexus.tar.gz`
NEXUSDIR=`echo $EXTOUT | cut -d '/' -f1`
rm -rf /tmp/nexus/nexus.tar.gz
rsync -avzh /tmp/nexus/ /opt/nexus/
sudo useradd nexus
sudo chown -R nexus.nexus /opt/nexus 
cat <<EOT>> /etc/systemd/system/nexus.service
[Unit]                                                                          
Description=nexus service                                                       
After=network.target                                                            
                                                                  
[Service]                                                                       
Type=forking                                                                    
LimitNOFILE=65536                                                               
ExecStart=/opt/nexus/$NEXUSDIR/bin/nexus start                                  
ExecStop=/opt/nexus/$NEXUSDIR/bin/nexus stop                                    
User=nexus                                                                      
Restart=on-abort                                                                
                                                                  
[Install]                                                                       
WantedBy=multi-user.target                                                      
EOT
echo 'run_as_user="nexus"' > /opt/nexus/$NEXUSDIR/bin/nexus.rc
sudo systemctl daemon-reload
sudo systemctl start nexus
sudo systemctl enable nexus

```

## SonarQube Setup

```
sudo apt update
sudo apt install net-tools unzip vim curl
```

Increase the virtual memory kernel with the maximum number of open files and resource limits.

```
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

You can make the changes persistent by modifying system parameters in the /etc/sysctl.conf configuration file.

```
sudo vim /etc/sysctl.conf
```

Add the following lines.

```
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

Save and exit. 

Open the ```limits.conf``` file.

```
sudo vim /etc/security/limits.conf
```
At the very bottom, add the following lines

```
sonarqube - nofile 65536
sonarqube - nproc 4096
```

Save and exit. For the changes to come into effect, reboot your server.


###### Install JDK

```
sudo apt install -y openjdk-11-jdk
```

###### Install PostgreSQL datbase

Download and add the PostgreSQL GPG key.

```
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```

Add the PostgreSQL repository.

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
```

Then update the package index to sync the new repository.

```
sudo apt update
```

Install the PostgreSQL database and its dependencies.

```
sudo apt install -y postgresql posgresql-contrib
```


By default, the PostgreSQL service gets started after installation, if not started run the following command.

```
sudo systemctl start postgresql
```

```
sudo systemctl status postgresql
```

Enable PostgreSQL to automatically start upon booting.

```
sudo systemctl enable postgresql
```


###### Configure PostgreSQL

Moving on, we are going to set the password for the Postgres user that usually comes by default when PostgreSQL is installed. To do so, run the command.

```
sudo passwd postgres
```

Type the password and confirm it. Next, switch to the Postgres user.

```
su - postgres
```

Next, proceed and create a new database user.

```
createuser sonar
```

Once done, switch to the PostgreSQL prompt using the command.

```
psql
```

With access to the PostgreSQL shell, create a password for the user you just created.

```
ALTER USER sonar WITH ENCRYPTED PASSWORD 'sonar';
```

Next, create a SonarQube database with the user you created as the owner

```
CREATE DATABASE sonarqube OWNER sonar;
```

Then, assign or grant all privileges to the database use such that they have all the privileges to modify the database.

```
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
```

Now exit the database.

```
\q
```


###### Download and configure SonarQuge

https://www.sonarqube.org/downloads/

To download the zip file, issue the command.

```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
```

Next, unzip the zipped file.

```
unzip sonarqube-9.4.0.54424.zip
```

And move it to the /opt/ path.

```
sudo mv sonarqube-9.4.0.54424 /opt/sonarqube
```

###### Create new user and group

Create a new user and group that will run the SonarQube service.

```
sudo groupadd sonar
```

Create the user with the home directory set to /opt/sonarqube as you add the user to the newly created group.

```
sudo useradd -c "SonarQube - User" -d /opt/sonarqube/ -g sonar sonar 
```

Then set ownership to the /opt/sonarqube directory.

```
sudo chown -R sonar:sonar /opt/sonarqube/
```


###### Configure SonarQube

Open the SonarQube configuration file.

```
sudo vim  /opt/sonarqube/conf/sonar.properties
```

Uncomment the following lines.

```
sonar.jdbc.username=<username>
sonar.jdbc.password=<password>
```

Modify these lines so that they look as what is provided.

```
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

```
sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError
```

Modify the following lines.

```
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.web.javaAdditionalOpts=-server
sonar.log.level=INFO
sonar.path.logs=logs
```

Modify the user that will run the SonarQube service by editing the file shown.

```
sudo vim /opt/sonarqube/bin/linux-x86-64/sonar.sh
```

Scroll down and ensure the line below appears as shown.

```
RUN_AS_USER=sonar
```


###### Create a systemd service file for SonarQube.

```
sudo vim  /etc/systemd/system/sonarqube.service
```

Add the following lines.

```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Group=sonar
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

```

Save the changes and exit the file.

Enable the SonarQube service to start upon booting.

```
sudo systemctl enable sonarqube
```

And start the SonarQube service.

```
sudo systemctl start sonarqube
```

To ensure that the SonarQube service is running, execute the command.

```
sudo systemctl statu sonarqube
```

