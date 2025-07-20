**Introduction**
In this tutorial, we will demonstrate how to download, install, start and stop a JBoss 7.1.1.Final server on CentOS 6.x. We use OpenJDK 6 for this tutorial.
This tutorial consists of the following steps


**Step 1:** JDK installation and verification


**Step 2:** Download JBoss and the installation procedure


**Step 3:** Create the appropriate user


**Step 4:** Start our new JBoss server and verify that the server has started properly


**Step 5:** Stop the new JBoss server and verify that the server has shutdown properly


**Step 1:** JDK Installation and verification**


The first step before installing JBoss AS 7, is to install a JDK. Any JDK can be used, such as Sun JDK, OpenJDK, IBM JDK, or JRocket etc. We chose Open JDK 6 for this tutorial, because it is the new Java reference implementation starting with Java 7.


**NOTE:** JDK 7 and above can also be used with JBoss. A JRE is also sufficient to run JBoss 7, however a JRE does not include some of the additional feature of a JDK.


**Installing OpenJDK:**
Issue the following command to install the JDK:
```bash
$ su -c "yum install java-1.6.0-openjdk-devel"
```


**Confirming the install:**
Issue the following command to confirm that the proper version of the JDK is on your classpath:
```bash
$ java -version
```
**NOTE:** For our installation, we are not defining a explicit JAVA_HOME for JBoss AS 7. The default works in this situation, because we don’t have multiple java versions installed. For most production environments with multiple versions of Java, it is recommended to set the JAVA_HOME in the standalone.conf or domain.conf files.


**Step 2:** Download JBoss and the installation procedure

The next step is to download the appropriate version of JBoss AS 7. We will download the .zip version of JBoss AS 7, and install it using the unzip utility.

Downloading JBoss AS 7.1.1.Final:

Issue the following wget to download jboss-as-7.1.1.Final.zip:

```bash
wget http://download.jboss.org/jbossas/7.1/jboss-as-7.1.1.Final/jboss-as-7.1.1.Final.zip
```

**NOTE:** jboss-as-7.1.1.Final.zip can also be downloaded with your favorite browser from the **http://www.jboss.org/jbossas/downloads/** page.

Installing JBoss AS 7.1.1.Final:

Next, we issue the following unzip command to finally install jboss-as-7.1.1.Final in the /usr/share directory:

```bash
$ unzip jboss-as-7.1.1.Final.zip -d /usr/share
```
Alternatively, any directory can be chosen for the JBoss 7 installation.


**Step 3:** Create the appropriate user**
Now that JBoss AS 7, is installed, we need to make sure that we create a user with the appropriate privileges. 

It is never a good idea to run JBoss as root for various reasons.

Create the new user:

We create a new user called jboss by issuing the following command:

```bash
$ adduser jboss
```

Alternatively, any username can be used.

Change ownership of the installation directory:

We need to assign the appropriate ownership to the installation directory for the newly created jboss user by issuing the command:

```bash
$ chown -fR jboss.jboss /usr/share/jboss-as-7.1.1.Final/
```
Switch user to the jboss user:

We switch to the jBoss user, so that this new installation can be administered properly. 

It is not recommended to administer JBoss as root.
```bash
$ su jboss
```
Change directory to the jboss bin directory:
Now, lets change directories to the JBoss bin directory. 

This dorectory contains the necessary scripts to start, stop and manage your JBoss installation.
```bash
$ cd /usr/share/jboss-as-7.1.1.Final/bin/
```
Add a jboss management user:
The final step before we start JBoss, is to add a management user. 

This is an internal JBoss management user, necessary to access the new JBoss management console.
```bash
$ ./add-user.sh
```
You should see the following message on the console after executing the command:
```bash
What type of user do you wish to add?

a) Management User (mgmt-users.properties)
b) Application User (application-users.properties)

(a): a
```
We select “a”, next you should see the following message:

Enter the details of the new user to add.

Realm (ManagementRealm) :

Username : jboss

Password :

Re-enter Password :

* hit enter for Realm to use default, then provide a username and password

We select the default value for the Realm (ManagementRealm), by hitting enter, and select “jboss” as our username. 

By default, we supply “jb0ss” as our password, of course, you can provide any password you prefer here.


**Step 4:** Start the JBoss AS 7 server:**

Once the appropriate JBoss users are created, we are now ready to start our new JBoss AS 7 server. 

With JBoss AS 7, a new standalone and domain model has been introduced. 

In this tutorial, we focus on starting up a standalone server. The domain server will be part of a future tutorial.

**Startup a JBoss 7, standalone instance:**

A standalone instance of JBoss 7 can be starting by executing:

```bash
$ ./standalone.sh -Djboss.bind.address=0.0.0.0 -Djboss.bind.address.management=0.0.0.0&;
```


**NOTE:** By default, JBoss 7 will only bind to localhost. 

This does not allow any remote access to your jboss server. 

We define the jboss.bind.address property as 0.0.0.0 and jboss.bin.address.management property to 0.0.0.0 as well. 

This allows us to access the remote JBoss. We could have also defined the hostname or the ip address. 

However, unless an elastic ip is used, this value can change. This is why we opted for 0.0.0.0.

You should see the following messages on the console after executing the command for a successful startup:

```bash 
=========================================================================
JBoss Bootstrap Environment
JBOSS_HOME: /usr/share/jboss-as-7.1.1.Final
JAVA: java
JAVA_OPTS: -server -XX:+UseCompressedOops -XX:+TieredCompilation -Xms64m -Xmx512m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true -Dorg.jboss.resolver.warning=true -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000 -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -Djboss.server.default.config=standalone.xml
=========================================================================
17:51:31,102 INFO [org.jboss.modules] JBoss Modules version 1.1.1.GA
17:51:31,501 INFO [org.jboss.msc] JBoss MSC version 1.0.2.GA
17:51:31,602 INFO [org.jboss.as] JBAS015899: JBoss AS 7.1.1.Final “Brontes” starting
17:51:33,705 INFO [org.xnio] XNIO Version 3.0.3.GA
17:51:33,721 INFO [org.jboss.as.server] JBAS015888: Creating http management service using socket-binding (management-http)
17:51:33,755 INFO [org.xnio.nio] XNIO NIO Implementation Version 3.0.3.GA
17:51:33,795 INFO [org.jboss.remoting] JBoss Remoting version 3.2.3.GA
17:51:33,891 INFO [org.jboss.as.logging] JBAS011502: Removing bootstrap log handlers
17:51:33,914 INFO [org.jboss.as.configadmin] (ServerService Thread Pool — 26) JBAS016200: Activating ConfigAdmin Subsystem
17:51:33,949 INFO [org.jboss.as.clustering.infinispan] (ServerService Thread Pool — 31) JBAS010280: Activating Infinispan subsystem.
17:51:34,077 INFO [org.jboss.as.webservices] (ServerService Thread Pool — 48) JBAS015537: Activating WebServices Extension
17:51:34,081 INFO [org.jboss.as.security] (ServerService Thread Pool — 44) JBAS013101: Activating Security Subsystem
17:51:34,105 INFO [org.jboss.as.security] (MSC service thread 1-1) JBAS013100: Current PicketBox version=4.0.7.Final
17:51:34,121 INFO [org.jboss.as.osgi] (ServerService Thread Pool — 39) JBAS011940: Activating OSGi Subsystem
17:51:34,128 INFO [org.jboss.as.naming] (ServerService Thread Pool — 38) JBAS011800: Activating Naming Subsystem
17:51:34,501 INFO [org.jboss.as.connector] (MSC service thread 1-1) JBAS010408: Starting JCA Subsystem (JBoss IronJacamar 1.0.9.Final)
17:51:34,583 INFO [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool — 27) JBAS010403: Deploying JDBC-compliant driver class org.h2.Driver (version 1.3)
17:51:34,944 INFO [org.jboss.as.naming] (MSC service thread 1-2) JBAS011802: Starting Naming Service
17:51:35,043 INFO [org.jboss.ws.common.management.AbstractServerConfig] (MSC service thread 1-1) JBoss Web Services – Stack CXF Server 4.0.2.GA
17:51:35,844 INFO [org.apache.coyote.http11.Http11Protocol] (MSC service thread 1-1) Starting Coyote HTTP/1.1 on http–0.0.0.0-8080
17:51:35,886 INFO [org.jboss.as.mail.extension] (MSC service thread 1-2) JBAS015400: Bound mail session [java:jboss/mail/Default]
17:51:36,543 INFO [org.jboss.as.remoting] (MSC service thread 1-1) JBAS017100: Listening on /0.0.0.0:9999
17:51:36,637 INFO [org.jboss.as.server.deployment.scanner] (MSC service thread 1-1) JBAS015012: Started FileSystemDeploymentService for directory /usr/share/jboss-as-7.1.1.Final/standalone/deployments
17:51:36,791 INFO [org.jboss.as.remoting] (MSC service thread 1-1) JBAS017100: Listening on /0.0.0.0:4447
17:51:36,811 INFO [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-2) JBAS010400: Bound data source [java:jboss/datasources/ExampleDS]
17:51:36,895 INFO [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://0.0.0.0:9990
17:51:36,896 INFO [org.jboss.as] (Controller Boot Thread) JBAS015874: JBoss AS 7.1.1.Final “Brontes” started in 6182ms – Started 133 of 208 services (74 services are passive or on-demand)
```

**Test your JBoss 7 installation:**

A good indication of a successful startup is that you can login to the JBoss admin console. http://yourip:9990/

This should provide you access to the new admin console, which will be the topic of a future tutorial.


**Step 5:** Stop the JBoss AS 7 server:

After successfully starting up JBoss 7, lets demonstrate how to shut your JBoss server down in this section.

**Shutdown a JBoss 7 instance:**
To shutdown your JBoss 7 server, execute the following command:
```bash
$ ./jboss-cli.sh --connect command=:shutdown
```
