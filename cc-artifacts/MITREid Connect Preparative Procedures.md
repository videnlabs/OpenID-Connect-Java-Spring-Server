# Preparative Procedures
## Base Operating System Install
1. Install Ubuntu LTS 20.04 on a virtual machine or a physical machine
## Install Java Development Kit (JDK)
1. Install java using ```sudo apt-get install default-jdk```
2. Validate java installation using ```java --version```
## Install Apache Maven
1. Download the archive using 
```
#wget https://dlcdn.apache.org/maven/maven-3/3.8.7/binaries/apache-maven-3.8.7-bin.tar.gz
```
NOTE: Refer to https://maven.apache.org/download.cgi for the latest version of Maven
2. Extract the distribution using ```tar xzvf apache-maven-3.8.7-bin.tar.gz```
3. Add the ```bin``` directory of the created directory ```apache-maven-3.8.7``` to the ```PATH``` environment variable by adding ```export PATH="<PATH_TO_EXTRACT>/apache-maven-3.8.7/bin:$PATH``` to ```~/bash.rc``` and executing ```source ~/.bashrc```
4. Confirm addition by running ```mvn -v```
## Install Apache Ant
1. Download tarball using ```wget https://dlcdn.apache.org//ant/binaries/apache-ant-1.9.16-bin.tar.gz```
2. Extract the distribution using ```tar xzvf apache-ant-1.9.16-bin.tar.gz```
3. Add the ```bin``` directory of the created directory ```apache-ant-1.9.16``` to the ```PATH``` environment variable by adding ```export PATH="<PATH_TO_EXTRACT>/ant-1.9.16/bin:<PATH_TO_EXTRACT>/apache-maven-3.8.7/bin:$PATH``` to ```~/bash.rc``` and executing ```source ~/.bashrc```
4. Create environmental variable ANT_HOME to <PATH_TO_EXTRACT>/apache-ant-1.9.16/ by adding ```export ANT_HOME=<PATH_TO_EXTRACT>/apache-ant-1.9.16/``` to .bashrc
## Install Apache Tomcat
1. Create a user group for Tomcat
```
# groupadd tomcat
# useradd -s /bin/false -g tomcat -d /home/tomcat tomcat
```
2. Download Tomcat using ```wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.84/bin/apache-tomcat-8.5.84.tar.gz```
3. Extract Tomcat using ```tar xzvf apache-tomcat-8.5.84.tar.gz```
4. Build Tomcat using Apache Ant
```
# cd apache-tomcat-8.5.84
# ant
```
5. Follow instructions in RUNNING.txt
## Install nginx reverse proxy
1. install nginx with ```sudo apt-get install nginx```
## Install MITREid Connect

# Example Setup Script
```
# Install java and nginx
apt-get install default-jdk nginx
java --version
wget https://dlcdn.apache.org/maven/maven-3/3.8.7/binaries/apache-maven-3.8.7-bin.tar.gz
tar xzvf apache-maven-3.8.7-bin.tar.gz
echo '$PATH="~/apache-maven-3.8.7/bin:~/apache-ant-1.6.19/bin:$path' >> .bashrc
echo '$ANT_HOME="~/apache-ant-1.6.19' >> .bashrc
source ~/bash.rc
wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.84/bin/apache-tomcat-8.5.84.tar.gz
cd apache-tomcat-8.5.4
./setup.sh
#Generate cert and key
# Create NGINX rule for Tomcat (forward 8080 to 443)
'upstream tomcat {
 server 127.0.0.1:8080 weight=100 max_fails=5 fail_timeout=5;
 }
 
server {
    listen              443 ssl;
    server_name         mitreid-connect.corp.viden.com;
    ssl_certificate     mitreid-connect.crt;
    ssl_certificate_key mitreid-connect.key;
    ssl_protocols       TLSv1.1 TLSv1.2;
    ssl_ciphers         EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;

location / {
 proxy_set_header X-Forwarded-Host $host;
 proxy_set_header X-Forwarded-Server $host;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_pass http://tomcat/;
}
'
git clone https://github.com/videnlabs/OpenID-Connect-Java-Spring-Server.git
cd Open-ID-Connect-Java-Spring-Server
mvn package
cp openid-connect-server-webapp/target/openid-connect-server-webapp.war var/lib/tomcat8/webapps
```
