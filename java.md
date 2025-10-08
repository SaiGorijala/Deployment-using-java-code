Java EE Application Deployment Guide: Build Server to Deploy Server
Workflow 1. Project Context and Goal This guide details the process of
creating, building, and deploying the classic JBoss/WildFly "Member
Registration" quickstart application (the one shown in your screenshot)
using a two-basic server model with minimal instance setup: a dedicated
Build Server and a dedicated Deployment Server. Server Build Server

Deploy Server

Role Compiles source code, runs tests, and produces the deployable
artifact (.war). Hosts the application server (WildFly/JBoss) and runs
the application.

Basic Ubuntu server with a t3.micro instance type

Key Components JDK, Apache Maven, Git

JDK, WildFly/JBoss Application Server

This is the final instance setup

2.  Prerequisites and Environment Setup Ensure the following tools are
    installed on both servers (or your local machine, if simulating
    servers using different directories/ports).Make sure that before
    installing the tools update the servers using command
    `sudo apt -y update`

Prerequisites (Both Servers) 1.​ Java Development Kit (JDK 8 or newer):
Must be installed in build server and configured correctly.

2.​ Maven: should also be installed in the build server only once the
compatible java version is installed so that maven wen installs
configures its settup according

3.​ WildFly or JBoss AS 7/EAP: Download and extract the application
server distribution. For this project, WildFly 10+ is the modern
equivalent of JBoss AS 7.

4.​ Networking: Ensure both servers can communicate on the necessary
ports (typically port 8080 for HTTP access and port 9990 for
management).

Setting up the Servers (Logical Separation) To simulate the two-server
environment on a single machine (if required), follow these steps: 1.​
Define Server Directories: ○​ Build Server Directory:
\~/javaee-build-server/ ○​ Deploy Server Directory:
\~/wildfly-deploy-server/ (This is where you extract the WildFly/JBoss
zip file). 2.​ Run Deploy Server (WildFly): Start the application server
on the Deploy Server. We will use the default standalone.xml
configuration.​ \# On the Deploy Server machine (or inside the extracted
WildFly directory)​ \$ cd \~/wildfly-deploy-server/bin​ \$ ./standalone.sh
​

3.  Project Creation and Build (On Build Server) All steps in this
    section are performed exclusively on the Build Server.

Step 3.1: Get the Project Source Code The project is based on the famous
JBoss "kitchensink" quickstart. It can be initialized via Maven or
cloned directly. We will clone a common repository structure. \# 1.
Navigate to the Build Server directory​ \$ cd \~/javaee-build-server/​ ​ \#
2. Clone the representative quickstart project​ $git clone
[https://github.com/jboss-developer/jboss-as-quickstarts.git$\](https://github.com/jboss-devel
oper/jboss-as-quickstarts.git\$) cd jboss-as-quickstarts/kitchensink​

Step 3.2: Configure and Inspect the Project The project is a standard
Java EE 6 application that uses JPA (for data), Bean Validation (for
constraints), JAX-RS (for a REST API), and JSF/Facelets (for the web
UI). ●​ Key Files: ○​
src/main/java/org/jboss/as/quickstarts/kitchensink/model/Member.java:
Contains the JPA entity with Bean Validation annotations (e.g.,
@NotNull, @Size, @Email). ○​ src/main/resources/import.sql: Used by JPA
(Hibernate) to populate initial data, including the "John Smith" record
visible in the screenshot. ○​ src/main/webapp/index.xhtml: The
JSF/Facelets view containing the registration form and member list.

Step 3.3: Build the Artifact (.war file) Use Apache Maven to compile the
source code, run any unit tests, and package the final deployable
artifact. \# On the Build Server​ \$ mvn clean package​ Output: Maven will
execute the lifecycle, and upon successful completion, it will generate
the deployable file. You should see a line similar to: \[INFO\] Building
war: .../jboss-as-quickstarts/kitchensink/target/kitchensink.war​ The
file kitchensink.war is now the final build artifact, residing in the
target/ directory of the Build Server.

4.  Transfer and Deployment (To Deploy Server) This is the critical step
    where the built artifact is moved from the Build Server to the
    Deploy Server.

Step 4.1: Transfer the Artifact We will use the Secure Copy Protocol
(scp) as a standard, secure way to transfer the file across a network.
This command needs to be executed on the Build Server. Assumptions: ●​
Deploy Server IP: 192.168.1.100 (Replace with the actual IP address of
your Deploy Server). ●​ Deploy Server User: deploy_user ●​ Deployment
Directory Path: The folder that WildFly/JBoss constantly monitors for
new application files is standalone/deployments/. \# On the Build Server​
$ARTIFACT_PATH=target/kitchensink.war$ DEPLOY_USER=deploy_user​

$DEPLOY_SERVER_IP=192.168.1.100$
REMOTE_DEPLOY_DIR=/path/to/wildfly-deploy-server/standalone/deployments/​
​\# Execute the transfer command​ \$ scp \$ARTIFACT_PATH
$DEPLOY_USER@$DEPLOY_SERVER_IP:\$REMOTE_DEPLOY_DIR​

Step 4.2: Automatic Deployment on Deploy Server As soon as the
kitchensink.war file is successfully placed into the
standalone/deployments/ folder on the Deploy Server, WildFly/JBoss
detects the new file and automatically attempts to deploy it. In the
WildFly console (the terminal where you ran ./standalone.sh), you will
see log messages confirming the deployment: \[org.jboss.as.server\]
(ServerService Thread Pool -- 29) WFLYSRV0010: Deployed
"kitchensink.war" (runtime-name : "kitchensink.war")​ If the deployment
is successful, WildFly creates a marker file named
kitchensink.war.deployed.

5.  Verification To verify that the deployment was successful and
    matches your screenshot, open a web browser and navigate to the
    Deploy Server's IP address and the application's context root. 1.​
    Access URL:​ http://`<DEPLOY_SERVER_IP>`{=html}:8080/kitchensink/​ ○​
    (Replace `<DEPLOY_SERVER_IP>`{=html} with localhost if running on
    the same machine, or the actual IP address otherwise). 2.​ Expected
    Output:​ You will see the Member Registration page, which includes
    the section: ○​ Register (Bean Validation example) ○​ A form to
    register a new member. ○​ A list displaying the initial data from
    import.sql (e.g., John Smith). The successful display of this page
    confirms that: ●​ The Build Server successfully compiled the
    artifact. ●​ The artifact was correctly transferred and deployed to
    the Deploy Server. ●​ All Java EE components (CDI, JPA, Bean
    Validation, JAX-RS) are running correctly.

6.  Alternative Deployment: JBoss Management CLI For a more automated,
    scriptable deployment, you can use the JBoss/WildFly Command Line
    Interface (CLI). This is considered a best practice for production
    environments. The CLI allows the Build Server to directly connect to
    the running Deploy Server and issue deployment commands. 1.​ Start
    CLI from the Build Server:​

(This assumes the Build Server has the WildFly client utilities
available, or you use a remote connection tool)​ \# Connect to the Deploy
Server's management port (9990)​ \# The management user must be created
using the 'add-user.sh' script first.​ \$ cd
/path/to/wildfly-deploy-server/bin​ \$ ./jboss-cli.sh --connect
--controller=192.168.1.100:9990​ 2.​ Deploy the Artifact:​ \# Inside the
CLI​ \[standalone@192.168.1.100:9990 /\] deploy
/path/to/kitchensink/target/kitchensink.war --name=kitchensink.war
--force​ \[standalone@192.168.1.100:9990 /\] quit​ This method is cleaner
as it manages the deployment lifecycle without relying on filesystem
copy triggers.


