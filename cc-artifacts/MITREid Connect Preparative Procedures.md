# Preparative Procedures
## Base Operating System Install
1. Install Ubuntu LTS 20.04 on a virtual machine or a physical machine
## Install Java Development Kit (JDK)
1. Install java using ```sudo apt-get install default-jdk```
2. Validate java installation using ```java --version```
3. Set JAVA_HOME using ```export JAVA_HOME=jvm/java-11-openjdk-amd64```
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
2. Extract the distribution using ```tar xzvf apache-ant-1.10.13-bin.tar.gz```
3. Add the ```bin``` directory of the created directory ```apache-ant-1.10.13``` to the ```PATH``` environment variable by adding ```export PATH="<PATH_TO_EXTRACT>/apache-ant-1.10.13/bin:<PATH_TO_EXTRACT>/apache-maven-3.8.7/bin:$PATH``` to ```~/bash.rc``` and executing ```source ~/.bashrc```
4. Create environmental variable ANT_HOME to <PATH_TO_EXTRACT>/apache-ant-1.10.13/ by adding ```export ANT_HOME=<PATH_TO_EXTRACT>/apache-ant-1.10.13/``` to .bashrc
5. Confirm installation using ```ant -version```

## Install nginx reverse proxy and certificates
### TODO: Fix NGINX config to forward to localhost:8080/openid-connect-server-webapp
Gor testing purposes, self-signed certificates can be used, but it is suggested that Let's Encrypt is used for externally accessivle instances.

1. Install nginx with ```sudo apt-get install nginx```
2. Create an intial configuration file the definition for the tomcat server in nginx (note: in this instance, mitre-connect.corp.viden.com is used as an example. Subsitute with your domain)

**/etc/nginx/sites-available/mitre-connect.corp.viden.com.conf**
```
server {
      server_name mitre-connect.corp.viden.com www.mitre-connect.corp.viden.com;
      }
```
 3. Restart nginx using ```sudo systemctl reload nginx```
 4. Enable the ufw firewall and configure ton allow Nginx Full
```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```
 6. Create a certificate with OpenSSL ```sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt```
 7. Create a Diffie-Hellman group for Perfect Forward Secrecy ```sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048```
 8. Create a snippet for the certificates
 **/etc/nginx/snippets/self-signed.conf**
 ```
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
 ```
 9. Create a snippet to enable encryption settings
 **/etc/nginx/snippets/ssl-params.conf**
 ```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable preloading HSTS for now.  You can use the commented out header line that includes
# the "preload" directive if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;

ssl_dhparam /etc/ssl/certs/dhparam.pem;
```
10. Configure default file
**/etc/nginx/available-sites/default**
```
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;
    server_name 10.0.40.101;
    return 302 https://$server_name$request_uri;
}

server {

    # SSL configuration

    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    include snippets/self-signed.conf;
    include snippets/ssl-params.conf;
    }
```
TODO: Copy in ubuntu from my local to /etc/nginx/sites-available
 11:wq
 12. Enable the ufw firewall and configure to allow Nginx Full
 13. Restart nginx using ```sudo systemctl reload nginx```  
      
## Install MITREid Connect and Start
1. Copy repo with ```git clone https://www.github.com/videlbabs/OpenID-Connect-Java-SpringServer.git```
2. Modify the file in ```openid-connect-server-webapp/src/main/webapp/WEB-INF/server-config.xml``` and change issuer to be the full URL as configured for NGINX. In this example, we will change 'http://localhost:8080/openid-connect-server-webapp' to 'https://10.0.20.17/openid-connect-server-webapp' 
Further info at  https://github.com/mitreid-connect/OpenID-Connect-Java-Spring-Server/wiki/Server-configuration
3. Navigate to OpenID-Connect-Java-SpringServer and execute ```mvn package``` to build the server
4. Start with Jetty
5. From parent directory
```
mvn clean install
```
5. From ```openid-connect-server-webapp``` folder
```
mvn jetty:run-war
```
6. Check the install has worked by navigating to https://localhost/openid-connect-server-webapp/ and login using values username: ```user``` and password: ```password```.

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
