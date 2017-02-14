# PCF TCP Routing And SSL
Use Pivotal Cloud Foundry's TCP routing feature to terminate SSL at your application

# Contents
Table of contents here

# Introduction
A common security requirement for customers in regulated industries such as banking and healthcare is that all traffic should be secured end-to-end with SSL. Prior to Pivotal Cloud Foundry 1.8, inbound SSL connections would always terminate on the Gorouter, and further encryption could only be achieved between the Gorouter and running applications by installing Pivotal's [IPsec Add-on](https://docs.pivotal.io/addon-ipsec/index.html)  

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

## Step 5 - Push the application to PCF Dev
    
    $ cd [GITHUB HOME]/gs-spring-boot/intial
    $ cf push gs-spring-boot -p target/gs-spring-boot-0.1.0.jar  

## Step 6 - Create a TCP route and map it to your application
    
    (Assuming we are in default pcfdev-space):
    $ cf create-route pcfdev-space tcp.local.pcfdev.io --random-port
    Creating route tcp.local.pcfdev.io for org pcfdev-org / space pcfdev-space as admin...
    OK
    Route tcp.local.pcfdev.io:61015 has been created
    $ cf map-route gs-spring tcp.local.pcfdev.io --port 61015

## Step 7 - Verify you can now connect directly to your application over SSL
Browse to https://tcp.local.pcfdev.io:61015/ (substitute your own port at the end):

View details of the certificate to verify it is the one you just generated:

# Further Reading
Enabling TCP Routing  
[http://docs.pivotal.io/pivotalcf/1-9/adminguide/enabling-tcp-routing.html](http://docs.pivotal.io/pivotalcf/1-9/adminguide/enabling-tcp-routing.html)  

How to tell application containers (running Java apps) to trust self-signed certs or a private/internal CA
[https://discuss.pivotal.io/hc/en-us/articles/223454928-How-to-tell-application-containers-running-Java-apps-to-trust-self-signed-certs-or-a-private-internal-CA](https://discuss.pivotal.io/hc/en-us/articles/223454928-How-to-tell-application-containers-running-Java-apps-to-trust-self-signed-certs-or-a-private-internal-CA)
