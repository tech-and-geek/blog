Hey guys hope you are doing well. Here is a new post regarding tomcat installation (multiple instances), hope you find this useful.


Let’s begin with the installation process:

**Check if the correct version of Java is installed**

JAVA is required for Tomcat to work,To check if java is installed on your server run:
```bash
# java -version
java version “1.7.0_55"
OpenJDK Runtime Environment (rhel-2.4.7.2.el7_0-x86_64 u55-b13)
OpenJDK 64-Bit Server VM (build 24.51-b03, mixed mode)
```
In case java is not installed on your system or you have version 1.6.x, you can install it by running:
```bash
# yum install java-1.7.0-openjdk.x86_64 java-1.7.0-openjdk-devel
```

** Download Tomcat**

You can find the latest version of  <a href="http://tomcat.apache.org/download-80.cgi">Tomcat at its download page</a>. You can download it with wget and extract it with tar like this:
```bash
# cd /usr/share
# wget http://www.apache.org/dist/tomcat/tomcat-8/v8.0.9/bin/apache-tomcat-8.0.9.tar.gz
# tar zxvf apache-tomcat-8.0.9.tar.gz
```
**Add tomcat user & group**

As it is not recommended to run Tomcat as root we will need to create an unprivileged user for it and set the apropiate owner of the tomcat folder:


```bash
# groupadd tomcat
# useradd -g tomcat -s /bin/bash -d /usr/share/apache-tomcat-8.0.9 tomcat
# chown -Rf tomcat.tomcat /usr/share/apache-tomcat-8.0.9/
```

**Running Tomcat**

To start tomcat we will first need to switch to the unprivileged user with:
```bash
# su – tomcat
```
And starting tomcat is as easy as running its startup script like this:
```bash
$ cd bin
$ ./startup.sh
The output should look like:
Using CATALINA_BASE: /usr/share/apache-tomcat-8.0.9
Using CATALINA_HOME: /usr/share/apache-tomcat-8.0.9
Using CATALINA_TMPDIR: /usr/share/apache-tomcat-8.0.9/temp
Using JRE_HOME: /
Using CLASSPATH: /usr/share/apache-tomcat-8.0.9/bin/bootstrap.jar:/usr/share/apache-tomcat-8.0.9/bin/tomcat-juli.jar
Tomcat started.
You should now be able to access it with a browser by either accessing http://localhost:8080 if this is a local computer or http://SERVER-IP:8080 if you are running it on a remote host.
To shutdown Tomcat you can simply run the shutdown script in the same folder like this:
$ ./shutdown.sh
```
** Setup user accounts**

Finally you have to configure Tomcat users so they can access admin/manager sections. You can do this by adding the users in the conf/tomcat-users.xml file with your favorite text editor. Add this text to the file:
```bash
<user username="manager" password="**PASSWORD**" roles="manager-gui" />
<user username="admin" password="**PASSWORD**" roles="manager-gui,admin-gui" />
```
**Running Multiple Instances of Tomcat (Optional)**

Sometimes you need to run more than one instance of Tomcat on the same server. To do this, as root, go back to the /usr/share directory where you first downloaded tomcat and extract it again in a different folder like this:
```bash
# cd /usr/share
# mkdir apache-tomcat-2
# tar zxvf apache-tomcat-8.0.9.tar.gz -C apache-tomcat-2 –strip-components 1
# chown -Rf tomcat.tomcat /usr/share/apache-tomcat-2/
```
Now we need to open the config/server.xml file in the new installation folder and change the port numbers like this:
The shutdown port from:
```bash
<Server port="8005" shutdown="SHUTDOWN">
```
to:
```bash
<Server port="8006" shutdown="SHUTDOWN">
```
The connector port from:
```bash
<Connector port="8080" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
```
to:
```bash
<Connector port="8081" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
```
And the AJP port from:
```bash
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```
to:
```bash
<Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
```
Now you can just switch back to the tomcat user and start the second instance like this:
```bash
# su – tomcat
$ cd /usr/share/apache-tomcat-2/bin/
$ ./startup.sh
Using CATALINA_BASE: /usr/share/apache-tomcat-2
Using CATALINA_HOME: /usr/share/apache-tomcat-2
Using CATALINA_TMPDIR: /usr/share/apache-tomcat-2/temp
Using JRE_HOME: /
Using CLASSPATH: /usr/share/apache-tomcat-2/bin/bootstrap.jar:/usr/share/apache-tomcat-2/bin/tomcat-juli.jar
Tomcat started.
```
You can now access the new instance of tomcat with your browser at **http://localhost:8081/**
