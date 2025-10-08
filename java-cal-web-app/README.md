# Building and Deploying a Java Web Calculator— Step-by-step Guide 
 
This document describes a practical, repeatable workflow to build a Java web application on one server (Build Server A) and deploy the generated WAR to a different server (App Server B) running Apache Tomcat. 

 
# Overview / Goal 

Build Server (A): where source code lives and you run mvn package to produce target/*.war. 
App Server (B): where Tomcat runs and where you deploy the WAR. 
This guide includes: prerequisites, exact commands, secure transfer methods, Tomcat deployment, automation, and troubleshooting. 

 

# Part A — Prepare Build Server

## 1. Install prerequisites 

update + essential tools 

sudo apt update 
sudo apt install -y openjdk-17-jdk maven git wget unzip curl 
sudo apt install -y maven 
 
## verify 

java -version 
mvn -version 

Use openjdk-11 if you need Java 11; adjust accordingly. 

 
<img width="921" height="276" alt="Image" src="https://github.com/user-attachments/assets/d13b26e6-525f-4f52-86c0-f027c9df73e4" />
 

## 2. Get the project source 

If your project is in Git: 

Git will be pre installed in the linux surver if nit install git using sudo apt –y install git than : 

```cd ~ git clone https://github.com/akracad/JavaWebCal.git ``` 

If the code is already on the build server, cd into the project directory: 

cd /home/ubuntu/JavaWebCalculator 

 
## 3. Check or update pom.xml (recommended) 
Ensure maven-war-plugin is modern version as when u install maven it installs the latest version: 

```
<build> 
  <plugins> 
    <plugin> 
      <groupId>org.apache.maven.plugins</groupId> 
      <artifactId>maven-war-plugin</artifactId> 
      <version>3.4.0</version> 
    </plugin> 
  </plugins> 
</build> 
```

Confirm maven-compiler-plugin target/source are correct for your Java version. 

## 4. Build the WAR from project root 

``` mvn clean package ```

maven comand will only wrok inside the project directory
 
## after success check 

``` ls -l target/*.war ```


 <img width="1797" height="294" alt="Image" src="https://github.com/user-attachments/assets/e7ed60be-a8c1-420c-a0ac-388b8a9eca1d" />


# Part B — Prepare App Deploy Server (Tomcat) 


## 1. Install Java (if not present) 

```
sudo apt update 
sudo apt install -y openjdk-17-jre-headless 
java -version

```

## 2. Install Tomcat on Deploy server 

``` cd /home/ubuntu ```

``` wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.109/bin/apache-tomcat-9.0.109.tar.gz ```

To extract the tar file 

``` tar -xvzf apache-tomcat-9.0.109.tar.gz ```

## make scripts executable 

``` sudo chmod +x /opt/tomcat/bin/*.sh ```
 
## shutdown 

```
Cd /opt/tomcat/bin/ 
./shutdown.sh
```
 
## start 

```
Cd /opt/tomcat/bin/ 
./startup.sh 

```


 <img width="1800" height="1025" alt="Image" src="https://github.com/user-attachments/assets/b30b2ced-162c-4285-8b9c-793b92ac83f5" />

 
## 3. Configure Tomcat Manager (if you want browser-managed deployments) 
Edit : vi /opt/tomcat/conf/tomcat-users.xml  and add a manager user: 

<role rolename="manager-gui"/> 
<role rolename="manager-script"/> 
<user username="admin" password="admin" roles="manager-gui,manager-script"/> 

 

# Part C — Transfer the WAR from Build server to Deploy Server


## scp using an SSH key local mechine Copy the SSH Key to Build Server
 

```
scp -i /path/to/local-key.pem /path/to/kk.key.pem ubuntu@BUILD_SERVER_IP:~/ 
```
 

## Transfer WAR from Build Server to Deploy Server
 
scp -i /path/to/SSH_KEY target/${WAR_NAME} ${DEPLOY_USER}@${DEPLOY_HOST}:${TOMCAT_HOME}/webapps/ 

Example substitution: 

scp -i kk.key.pem /home/ubuntu/JavaWebCalculator/target/webapp-0.1.3.war ubuntu@13.222.165.210:~/apache-tomcat-9.0.109/webapps/  

 

## After copying, SSH to deploy server confirm the file is present


``` Ls /apache-tomcat-9.0.109/webapps/  ```

 

You should see webapp-0.1.3.war in that path 

 <img width="1786" height="96" alt="Image" src="https://github.com/user-attachments/assets/c84f1b3a-b561-497c-8fb6-a3db83721611" />

 
# Part D — Deploy & Restart Tomcat 

1. If Tomcat auto-deploys (default) 
Copying the WAR into $TOMCAT_HOME/webapps/ is enough. Tomcat will unpack and deploy the WAR automatically. 
2. If Not just Restart the Tomcat

``` 
cd /home/ubuntu/apache-tomcat-9.0.109/bin 
./shutdown.sh  
./startup.sh

```

# Verify app is running


Open: http://${DEPLOY_HOST}:8080/ 
 

Example :  http://13.222.165.210:8080 

 
<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/8dfce7de-1d34-4779-94a1-4fd87895ebe9" />
 
