

{0}------------------------------------------------

# **Java EE Application Deployment Guide: Build Server to Deploy Server Workflow**

# **1. Project Context and Goal**

This guide details the process of creating, building, and deploying the classic JBoss/WildFly **"Member Registration"** quickstart application (the one shown in your screenshot) using a two-basic server model with minimal instance setup: a dedicated **Build Server** and a dedicated **Deployment Server**.

| Server           | Role                                  | Key<br>Components                    |
|------------------|---------------------------------------|--------------------------------------|
| Build<br>Server  | Compiles<br>source<br>code,<br>runs   | JDK,<br>Apache<br>Maven,<br>Git      |
|                  | tests,<br>and<br>produces<br>the      |                                      |
|                  | deployable<br>artifact<br>(.war).     |                                      |
| Deploy<br>Server | Hosts<br>the<br>application<br>server | JDK,<br>WildFly/JBoss<br>Application |
|                  | (WildFly/JBoss)<br>and<br>runs<br>the | Server                               |
|                  | application.                          |                                      |

Basic Ubuntu server with a t3.micro instance type

{1}------------------------------------------------

This is the final instance setup

# **2. Prerequisites and Environment Setup**

Ensure the following tools are installed on both servers (or your local machine, if simulating servers using different directories/ports).Make sure that before installing the tools update the servers using command

```
``` sudo apt -y update ```
```

## **Prerequisites (Both Servers)**

1. **Java Development Kit (JDK 8 or newer):** Must be installed in build server and configured correctly.

2. **Maven:** should also be installed in the build server only once the compatible java version is installed so that maven wen installs configures its settup according

{2}------------------------------------------------

```
ubuntu@ip-172-31-20-12:~$ sudo apt install maven
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
   libaopalliance-java libapache-pom-java libatinject-jsr330-api-java libcdi-api-java libcommo liberror-prone-java libgeronimo-annotation-1.3-spec-java libgeronimo-interceptor-3.0-spec-java libmayon-resolver-java libmayon-shared-utils-java libmayon-core-java libmleyus-cipher-java
```

3. **WildFly or JBoss AS 7/EAP:** Download and extract the application server distribution. For this project, **WildFly 10+** is the modern equivalent of JBoss AS 7.

```
| Ubuntu#ip-172-31-18-94:-$ wget https://download.jboss.org/wildfly/10.0.0.Final/wildfly-10.0.0.Final.zip
| -2025-10-08 07:04:33-- https://download.jboss.org/wildfly/10.0.0.Final/wildfly-10.0.0.Final.zip
| Resolving download.jboss.org (download.jboss.org)... 23.53.11.147, 23.53.11.141, 2500:1408:c400:e::17cd:6a11, ...
| Connecting to download.jboss.org (download.jboss.org)123.53.11.147|:443... connected.
| HTTP request sent, awaiting response... 200 OK | Length: 138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158431 (138158
```

```
ubuntu@ip-172-31-18-94:~$ sudo apt install zip
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
    unzip
```

```
[ubuntu@ip-172-31-18-94:~$ unzip wildfly-10.0.0.Final.zip
Archive: wildfly-10.0.0.Final.zip
    creating: wildfly-10.0.0.Final/
    creating: wildfly-10.0.0.Final/.installation/
    creating: wildfly-10.0.0.Final/appclient/
    creating: wildfly-10.0.0.Final/appclient/configuration/
    creating: wildfly-10.0.0.Final/bin/
    creating: wildfly-10.0.0.Final/bin/client/
    creating: wildfly-10.0.0.Final/docs/
```

```
[ubuntu@ip-172-31-18-94:~$ mv wildfly-10.0.0.Final ~/wildfly-10
[ubuntu@ip-172-31-18-94:~$ ls
wildfly-10 wildfly-10.0.0.Final.zip
ubuntu@ip-172-31-18-94:~$
```

{3}------------------------------------------------

4. **Networking:** Ensure both servers can communicate on the necessary ports (typically port 8080 for HTTP access).

## **Setting up the Servers (Logical Separation)**

To simulate the two-server environment on a single machine (if required), follow these steps:

- 1. **Define Server Directories:**
  - **Build Server Directory:** ~/javaee-build-server/
  - **Deploy Server Directory:** ~/wildfly-deploy-server/ (This is where you extract the WildFly/JBoss zip file).
- 2. **Run Deploy Server (WildFly):** Start the application server on the Deploy Server. We will use the default standalone.xml configuration.
  - # On the Deploy Server machine (or inside the extracted WildFly directory)
  - \$ cd ~/wildfly-deploy-server/bin
  - \$ ./standalone.sh

# **3. Project Creation and Build (On Build Server)**

All steps in this section are performed exclusively on the **Build Server**.

## **Step 3.1: Get the Project Source Code**

The project is based on the famous JBoss "kitchensink" quickstart. It can be initialized via Maven or cloned directly. We will clone a common repository structure.

# 1. Navigate to the Build Server directory

\$ cd ~/javaee-build-server/

{4}------------------------------------------------

# 2. Clone the representative quickstart project \$git clone

[https://github.com/jboss-developer/jboss-as-quickstarts.git\$](https://github.com/jboss-devel oper/jboss-as-quickstarts.git\$) cd jboss-as-quickstarts/kitchensink

### **Step 3.2: Configure and Inspect the Project**

The project is a standard Java EE 6 application that uses JPA (for data), Bean Validation (for constraints), JAX-RS (for a REST API), and JSF/Facelets (for the web UI).

#### ● **Key Files:**

- src/main/java/org/jboss/as/quickstarts/kitchensink/model/Member.java: Contains the JPA entity with Bean Validation annotations (e.g., @NotNull, @Size, @Email).
- src/main/resources/import.sql: Used by JPA (Hibernate) to populate initial data, including the "John Smith" record visible in the screenshot.
- src/main/webapp/index.xhtml: The JSF/Facelets view containing the registration form and member list.

#### **Step 3.3: Build the Artifact (.war file)**

Use Apache Maven to compile the source code, run any unit tests, and package the final deployable artifact.

# On the Build Server

\$ mvn clean package

#### Output:

{5}------------------------------------------------

# **4. Transfer and Deployment (To Deploy Server)**

This is the critical step where the built artifact is moved from the Build Server to the Deploy Server.

### **Step 4.1: Transfer the Artifact**

We will use the **Secure Copy Protocol (scp)** as a standard, secure way to transfer the file across a network. This command needs to be executed on the **Build Server**.

#### **Assumptions:**

- **Deploy Server IP:** 192.168.1.100 (Replace with the actual IP address of your Deploy Server).
- **Deploy Server User:** deploy\_user
- **Deployment Directory Path:** The folder that WildFly/JBoss constantly monitors for new application files is standalone/deployments/.
- **On the Build Server**

First get your pem file to build server from local machine

Than transfer the build file from the build server to the deploy server using scp -i

{6}------------------------------------------------

#### **Step 4.2: Automatic Deployment on Deploy Server**

As soon as the kitchensink.war file is successfully placed into the standalone/deployments/ folder on the Deploy Server, **WildFly/JBoss detects the new file and automatically attempts to deploy it.**

In the WildFly console (the terminal where you ran ./standalone.sh), you will see log messages confirming the deployment:

[org.jboss.as.server] (ServerService Thread Pool -- 29) WFLYSRV0010: Deployed "kitchensink.war" (runtime-name : "kitchensink.war")

If the deployment is successful, WildFly creates a marker file named kitchensink.war.deployed.

# **5. Verification**

To verify that the deployment was successful and matches your screenshot, open a web browser and navigate to the Deploy Server's IP address and the application's context root.

#### 1. **Access URL:**

http://<DEPLOY\_SERVER\_IP>:8080/kitchensink/

- (Replace <DEPLOY\_SERVER\_IP> with localhost if running on the same machine, or the actual IP address otherwise).
- 2. Expected Output:

You will see the Member Registration page, which includes the section:

- **Register (Bean Validation example)**
- A form to register a new member.
- A list displaying the initial data from import.sql (e.g., **John Smith**).

The successful display of this page confirms that:

- The Build Server successfully compiled the artifact.
- The artifact was correctly transferred and deployed to the Deploy Server.
- All Java EE components (CDI, JPA, Bean Validation, JAX-RS) are running correctly.

{7}------------------------------------------------

