## Instructions to install Jenkins.

For Debian based distributions.



First add the key to your system.

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
	/usr/share/keyrings/jenkins-keyring.asc > /dev/null
```




Add a Jenkins apt repository entry:

```
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
	https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
	/etc/apt/sources.list.d/jenkins.list > /dev/null
```




Update your local package index, then finally install Jenkins:
```
sudo apt update
sudo apt install fontconfig openjdk-11-jre
sudo apt install jenkins
```



For RedHat based distributions.
```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
	https://pkg.jenkins.io/redhat-stable/jenkins.repo


sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key


sudo yum upgrade
```

```
# add required dependencies for the jenkins package

sudo yum install java-11-openjdk

sudo yum install jenkins

sudo systemctl daemon-reload
```


Start Jenkins


You can enable the Jenkins service to start at boot time with the command.
```
sudo systemctl enable jenkins
```


You can start the Jenkins service with the command.
```
sudo systemctl start jenkins
```


You can check the status of the Jenkins service using the command.
```
sudo systemctl status jenkins
```




## Instructions to install Maven.


Using respective Package Manager.

For Debian based distributions.

	```
	sudo apt install -y maven
	```

For Red Hat based distributions.

	```
	sudo dnf install -y maven

	or

	sudo yum install -y maven
	```




## Instructions to install Apache Tomcat Application Server.


For Security purposes, Tomcat should run under a seperate, unprivalaged user.


Create a seperate user with name tomcat with appropriate permissions.

```
sudo useradd -m -d /opt/tomcat -U -s /bin/false tomcat
```
-m, --create-home 	- Create the user's home directory if it does not exist.

-d, --home-dir 		- The new user will be created using HOME_DIR as the value for the user's login directory.

-U, --user-group 	- Create a group with the same name as the user, and add the user to this group.

-s, --shell			- The name of a new user's login shell.

/bin/false			- as the user's default shell to ensure that it’s not possible to log in as tomcat.



Install OpenJDK.

	
Before installing JDK, first update the package manager cache.
```
sudo apt update
```

Install JDK by running the following command.

```
sudo apt install -y default-jdk
```
or 
```
sudo apt install -y openjdk-11-jdk
```



Download the archive using wget by running the following command.

```
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.21/bin/apache-tomcat-10.0.21.tar.gz
```

Then, extract the archive you downloaded by running.

```
sudo tar xzvf apache-tomcat-10*tar.gz -C /opt/tomcat --strip-components=1
```


Since you have already created a user, you can now grant tomcat ownership over the extracted installation by running

```
sudo chown -R tomcat:tomcat /opt/tomcat/

sudo chmod -R u+x /opt/tomcat/bin
```




Configuring Admin Users.


To gain access to the Manager and Host Manager pages, you’ll define privileged users in Tomcat’s configuration. You will need to remove the IP address restrictions, which disallows all external IP addresses from accessing those pages.




Tomcat users are defined in /opt/tomcat/conf/tomcat-users.xml. Open the file for editing with the following command:


```
sudo vim /opt/tomcat/conf/tomcat-users.xml
```


Add the following lines before the ending tag.

```
<role rolename="manager-gui" />
<user username="manager" password="manager_password" roles="manager-gui" />

<role rolename="admin-gui" />
<user username="admin" password="admin_password" roles="manager-gui,admin-gui" />
```


Here you define two user roles, manager-gui and admin-gui, which allow access to Manager and Host Manager pages, respectively. You also define two users, manager and admin, with relevant roles.

By default, Tomcat is configured to restrict access to the admin pages, unless the connection comes from the server itself. To access those pages with the users you just defined, you will need to edit config files for those pages.



To remove the restriction for the Manager page, open its config file for editing.

```
sudo vim /opt/tomcat/webapps/manager/META-INF/context.xml
```
```
<Context antiResourceLocking="false" privileged="true" >
	<CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
					sameSiteCookies="strict" />
<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
			allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
	<Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.Csr>
</Context>
```

Save and close the file, then repeat for Host Manager.

```
sudo vim /opt/tomcat/webapps/host-manager/META-INF/context.xml
```

You have now defined two users, manager and admin, which you will later use to access restricted parts of the management interface.



Create a systemd service.

```	
sudo update-java-alternatives -l
```


Note the path where Java resides, listed in the last column. You’ll need the path momentarily to define the service.





You’ll store the tomcat service in a file named tomcat.service, under /etc/systemd/system. Create the file for editing by running.

```
sudo vim /etc/systemd/system/tomcat.service
```
```
[Unit]
Description=Tomcat
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```


When you’re done, save and close the file.



Reload the systemd daemon so that it becomes aware of the new service.

```
sudo systemctl daemon-reload
```

You can then start the Tomcat service by typing.

```
sudo systemctl start tomcat
```

You can then check the status of Tomcat Server.

```
sudo systemctl status tomcat
```



To enable Tomcat starting up with the system, run the following command.

```
sudo systemctl enable tomcat
```


For Red Hat based distributions.


Install OpenJDK.

```
sudo dnf install java-11-openjdk -y
```
or 
```
sudo yum install java-11-openjdk -y
```


Install Apache Tomcat Application Server

```
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.21/bin/apache-tomcat-10.0.21.tar.gz
```

Untar downloaded binary	
```
tar -xvf apache-tomcat-10.0.21.tar.gz
```
```
mv apache-tomcat-10.0.21.tar.gz tomcat10\
```
```
mv tomcat10 /usr/local/
```



Create a user
```
sudo useradd -r tomcat
```

```
sudo chown -R tomcat:tomcat /usr/local/tomcat10
```


Create a tomcat.service unit file under /etc/systemd/system/ directory using your favorite text editor.

```
sudo vim /etc/systemd/system/tomcat.service
```
```
[Unit]
Description=Apache Tomcat Application Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=CATALINA_PID=/usr/local/tomcat10/temp/tomcat.pid
Environment=CATALINA_HOME=/usr/local/tomcat10
Environment=CATALINA_BASE=/usr/local/tomcat10

ExecStart=/usr/local/tomcat10/bin/catalina.sh start
ExecStop=/usr/local/tomcat10/bin/catalina.sh stop

RestartSec=10
Restart=always
[Install]
WantedBy=multi-user.target
```

Save and reload the System configuration.
```
sudo systemctl daemon-reload
```


Then start Tomcat service and enable it to auto start at system boot time.

```
sudo systemctl start tomcat
```
```
sudo systemctl enable tomcat
```
```
sudo systemctl status tomcat
```


Enable HTTP Authentication for Tomcat manager and Host manager

```
sudo vim /usr/local/tomcat10/conf/tomcat-users.xml
```

Add the following lines of code.
```
<role rolename="admin-gui,manager-gui"/> 
<user username="admin" password="tomhost@80" roles="admin-gui,manager-gui"/>
```

Enable Remote access to Tomcat manager and Host manager

```
sudo vim /usr/local/tomcat10/webapps/manager/META-INF/context.xml
```


```
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1 |.*" />
```


Allow Tomcat access from any host or network.



Restart Tomcat Service.

```
sudo systemctl restart tomcat
```



## Jenkins Automated Deployment


Step 1 :- Go to Manage Jenkins → Manage Plugins. Go to the Available section and find the plugin “Deploy to container Plugin” and install the plugin. Restart the Jenkins.

Step 2 :- Go to your Build project and click the Configure option. Choose the option “Deploy war/ear to a container.


Step 3 :- In the Deploy war/ear to a container section, enter the required details of the server on which the files need to be deployed and click on the Save button. These steps will now ensure that the necessary files get deployed to the necessary container after a successful build.