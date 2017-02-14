# Walkthrough: Cloud Foundry TCP Routing and SSL
Use Cloud Foundry's TCP routing feature to terminate SSL directly in your application

# Introduction
A common security requirement for customers in regulated industries such as banking and healthcare is that all traffic should be secured end-to-end with SSL. 

Prior to Pivotal Cloud Foundry 1.8, inbound SSL connections would [always terminate on the Gorouter](http://docs.pivotal.io/pivotalcf/1-9/adminguide/securing-traffic.html#ssl_options), and further encryption could only be achieved between the Gorouter and running applications by installing Pivotal's [IPsec Add-on](https://docs.pivotal.io/addon-ipsec/index.html)  

With the introduction in version 1.8 of TCP routing, it is now possible to terminate SSL right at your application - and this article will walk you through a working example of a Spring Boot application that is secured with SSL.  

# Prerequisites  
PCF Dev version 0.23.0 or later    
JDK 1.8 or later  
Gradle 2.3+ or Maven 3.0+  
git (tested on 2.10.1)   
A Linux-like environment (you will need to change the file paths for the directory commands to work on Windows)  

# How to do it
## Step 1 - Create a Spring Boot application  
We're going to be lazy here, and simply make a couple of small modifications to the Spring Boot Getting Started application:
    
    $ git clone https://github.com/spring-guides/gs-spring-boot.git

## Step 2 - Create an SSL certificate   
    
    $ cd [GITHUB HOME]/gs-spring-boot/intial/src/main/resources
    $ keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 \
    -keystore keystore.p12 -validity 3650 -keypass CHANGEME -storepass CHANGEME \
    -dname "C=GB,ST=Greater London,L=London,O=Dell EMC,OU=Apps and Data,CN=abd.dell.com"  
 
## Step 3 - Configure Spring Boot to use SSL and the new certificate  
(You can also retrieve the `application.properties` shown below from [here](https://github.com/bendalby82/cf-tcp-routing-ssl/blob/master/scripts/application.properties))  

    $ cd [GITHUB HOME]/gs-spring-boot/intial/src/main/resources
    $ cat <<EOT >> application.properties  
    server.port: 8080
    server.ssl.key-store: classpath:keystore.p12
    server.ssl.key-store-password: CHANGEME
    server.ssl.keyStoreType: PKCS12
    server.ssl.keyAlias: tomcat
    EOT  
    
## Step 4 - Package the application
    
    $ cd [GITHUB HOME]/gs-spring-boot/intial  
    $ mvn clean package  

## Step 5 - Push the application to PCF Dev (use default org and space)
    
    $ cd [GITHUB HOME]/gs-spring-boot/intial
    $ cf target -o pcfdev-org -s pcfdev-space
    $ cf push gs-spring-boot -p target/gs-spring-boot-0.1.0.jar  

## Step 6 - Create a TCP route and map it to your application
    
    $ cf create-route pcfdev-space tcp.local.pcfdev.io --random-port
    Creating route tcp.local.pcfdev.io for org pcfdev-org / space pcfdev-space as admin...
    OK
    Route tcp.local.pcfdev.io:61015 has been created
    $ cf map-route gs-spring tcp.local.pcfdev.io --port 61015

## Step 7 - Verify you can now connect directly to your application over SSL
Browse to https://tcp.local.pcfdev.io:61015/ (substitute your own port after the colon):  
<img src="https://github.com/bendalby82/cf-tcp-routing-ssl/blob/master/images/HTTPS-to-my-app.png" width="500px">  

View details of the certificate to verify it is the one you just generated (note the procedure has just changed if you are using [Chrome](http://www.howtogeek.com/292076/how-do-you-view-ssl-certificate-details-in-google-chrome/)):  
<img src="https://github.com/bendalby82/cf-tcp-routing-ssl/blob/master/images/view-certificate.png" width="650px">  

# Further Reading
**Enabling TCP Routing**  
[http://docs.pivotal.io/pivotalcf/1-9/adminguide/enabling-tcp-routing.html](http://docs.pivotal.io/pivotalcf/1-9/adminguide/enabling-tcp-routing.html)  

**How to tell application containers (running Java apps) to trust self-signed certs or a private/internal CA**
[https://discuss.pivotal.io/hc/en-us/articles/223454928-How-to-tell-application-containers-running-Java-apps-to-trust-self-signed-certs-or-a-private-internal-CA](https://discuss.pivotal.io/hc/en-us/articles/223454928-How-to-tell-application-containers-running-Java-apps-to-trust-self-signed-certs-or-a-private-internal-CA)  
  
**Enable HTTPS in Spring Boot**  
[https://drissamri.be/blog/java/enable-https-in-spring-boot/](https://drissamri.be/blog/java/enable-https-in-spring-boot/)
