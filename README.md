# Jenkins Installation and Setup
This Jenkins setup is based on the "Dockerfile.jenkins.openjdk" Docker file. This dockerfile uses openjdk as the base image, installs apache and downloads the latest jenkins.war file and deployes it. 

## How to build image
Download the Dockerfile.jenkins.openjdk and build the image.
$ docker build -t jenkins -f Dockerfile.jenkins.openjdk

## How to run
The above command should build a Docker image called, jenkins:latest. 
$ docker run -p 8080:8080 -p 50000:50000 --network=InfraNET --name jenkins -v $HOME/jenkins/jenkins_data:/var/jenkins_home jenkins

When you first start the jenkins container it will generate the jenkis admin password. Use that to login to Jenkins (localhost:8080).

### Note
1. I have mounted a host directory ($HOME/jenkins/jenkins_data) into the container. 
2. I am using a custom network called, InfraNET. This network was created as part of the Oracle 12c Fusion Middleware setup, which contains an Oracle 12.2 database, Oracle FMW 12c Admin and Managed Server containers as well. Making it part of the same network enables easy communication between the containers by just using the container name. 


## Post Container Setup (Dockerfile.jenkins.openjdk):
1. By default the container won't be able to run ant scripts, even if the plugin is enabled. To install apache-ant:
 - Login to the docker container as root
 $ docker container exec -it --user root jenkins bash
 - Install apache-ant
 $ apk add apache-ant
2. To be able to deploy ear files into Weblogic you need the weblogic deployment plugin installed and configured. 
   ### Note
   There is a bug using the wlfullclient.jar file generated using Oracle FMW 12.2, the work around is to use the wlfullclient.jar   
   generated using an FMW 10.3.6.

## Configure Jenkins
1. Login to jenkins and install the following plugins: Ant Plugin, Subversion Plug-in and Deploy Weblogic plugin.
2. To configure Weblogic Deploy Plugin:
   - Go to Manage Jenkins->Configure system, Under Weblogic Deployment Plugin set
    - Additional classpath: /var/jenkins_home/workspace/wlfullclient.jar -- This jar file was generated using WLS 10.3.6
    - Configuration file: /var/jenkins_home/workspace/weblogic-config.xml
    - Update the weblogic-config.xml as necessary (This file was manully generated)
      <?xml version="1.0" encoding="UTF-8"?>
      <config xmlns="http://org.jenkinsci.plugins/WeblogicDeploymentPlugin"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://org.jenkinsci.plugins/WeblogicDeploymentPlugin plugin-configuration.xsd">
      <weblogic-targets>
          <weblogic-target>
              <name>AdminServer</name>
              <host>InfraAdminContainer</host>
              <port>7001</port>
              <login>weblogic</login>
              <password>weblogic1234</password>
              <authMode>BY_LOGIN</authMode>
              <!-- optionnal 
              <ftpHost>_weblogic.remote.host_</ftpHost>
              <ftpUser>_weblogic.remote.user_</ftpUser>
              <ftpPassowrd>_weblogic.remote.password_</ftpPassowrd>
              <remoteDir>/remote/file/to/path/used/to/deploy/libraries</remoteDir>
              -->
          </weblogic-target>
      </weblogic-targets>
      </config>    
 3. To configure the weblogic deployment plugin inside the jenkins project.
    - Go to Mange Project->configure->post build actions, and set the following
     - Task Name:
     - Environment: AdminServer (InfraAdminContainer:7001)  -- This is automatically poplated reading the weblogic-config.xml file.
     - Name: P**M***App
     - Base directory where the resource to deploy can be found: /var/jenkins_home/workspace/SVN_Project/ant/P**M****App
     - Built resource to deploy: P**M****App.ear
     - Targets: infra_server1

- The current image does not have apache-ant install and as a result ant build scripts won't work. 
  - To install apache-ant, login to the docker container and run apk add apache-ant 
  - Copied the below wlfulclient.jar generated using the 10.3.6 image into the jenkins/jenkins_data/workspace folder.
 

## Additional notes:
1. Using the weblogic deployment plugin to push the builds to weblogic. I am using the WLS 12.2.1 image and the default weblogic deployment plugin that we generated using wls 12.2 does not work with jenkins.
 - Steps to build wlfullclient.jar using 12.2 (which did not work)
  - go to cd $WL_HOME/server/lib
  - java -jar wljarbuilder.jar (This generates the wlfullclient.jar file to be used inside Jenkins)
 - Solution: is to use the wlfullclient generated using WLS 10.3.6 
  - Steps to buld wlfullclient.jar using WLS 1036 image.
  - Downloaded 1036 docker image: docker pull fengzhou/docker-weblogic-1036
  - Run the above container image: docker container run --rm -it -v /home/docker/oracle/wls1036/:/tmp fengzhou/docker-weblogic-1036 bash
   - went to: cd /opt/bea/wls1036/weblogic10/server/lib/
   - build wlfulclient.jar: /opt/bea/jdk1.6.0_27/bin/java -jar wljarbuilder.jar
 
 2. Also enabled "enable tunneling" under the protocols section of the managed server from the console.


