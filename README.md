# jenkins


# Initial Setup:
# When you first start the jenkins container it will generate the jenkis admin password.
# This container is built using the: Dockerfile.jenkins.openjdk
#  - Uses openjdk as the base image, installs apache, downloands the latest jenkins.war file and deployes it.
#  - The current image does not have apache-ant install and as a result ant build scripts won't work. 
#  - To install apache-ant, login to the docker container and run apk add apache-ant 
#  - Copied the below wlfulclient.jar generated using the 10.3.6 image into the jenkins/jenkins_data/workspace folder.
# 
# Configure Jenkins (after login):
# - Install plugins: Ant Plugin, Deploy Weblogic Plugin, Subversion Plug-in
# - Go to Manage Jenkins-> Configure system -> Under WebLogic Deployment Plugin, set:
# - Additional classpath: /var/jenkins_home/workspace/wlfullclient.jar   [This is the wlfullclient.jar generated using WLS 10.3.6]
# - Configuration file: /var/jenkins_home/workspace/weblogic-config.xml
#   - This file has the following values and was manually generated 
#
#<?xml version="1.0" encoding="UTF-8"?>
#<config xmlns="http://org.jenkinsci.plugins/WeblogicDeploymentPlugin"
#xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
#xsi:schemaLocation="http://org.jenkinsci.plugins/WeblogicDeploymentPlugin plugin-configuration.xsd">
#<weblogic-targets>
#    <weblogic-target>
#        <name>AdminServer</name>
#        <host>InfraAdminContainer</host>
#        <port>7001</port>
#        <login>weblogic</login>
#        <password>weblogic1234</password>
#        <authMode>BY_LOGIN</authMode>
#        <!-- optionnal 
#        <ftpHost>_weblogic.remote.host_</ftpHost>
#        <ftpUser>_weblogic.remote.user_</ftpUser>
#        <ftpPassowrd>_weblogic.remote.password_</ftpPassowrd>
#        <remoteDir>/remote/file/to/path/used/to/deploy/libraries</remoteDir>
#        -->
#    </weblogic-target>
#</weblogic-targets>
#</config>
#
# - Then go to manage project->configure->post build actions, and set the following:
#   - Task Name: 
#   - Environment: AdminServer (InfraAdminContainer:7001)     -- This is automatically populated by reading the above weblogic-config.xml file.
#   - Name: P**M****App
#   - Base directory where the resource to deploy can be found: /var/jenkins_home/workspace/SVN_Project/ant/P**M****App
#   - Built resource to deploy: P**M****App.ear
#   - Targets: infra_server1
#
# To-Do: I need to update the Dockerifle to install apache-ant as well.
#
# Note:
# - Using the weblogic deployment plugin to push the builds to weblogic. I am using the WLS 12.2.1 image and 
#   the default weblogic deployment plugin that we generated using wls 12.2 does not work with jenkins.
#   - Steps to build wlfullclient.jar using 12.2 (which did not work)
#    - go to cd $WL_HOME/server/lib
#    - java -jar wljarbuilder.jar (This generates the wlfullclient.jar file to be used inside Jenkins)
# - Fix: is to use the wlfullclient generated using WLS 10.3.6 
#   - Steps to buld wlfullclient.jar using WLS 1036 image.
#   - Downloaded 1036 docker image: docker pull fengzhou/docker-weblogic-1036
#   - Run the above container image: docker container run --rm -it -v /home/docker/oracle/wls1036/:/tmp fengzhou/docker-weblogic-1036 bash
#   - went to: cd /opt/bea/wls1036/weblogic10/server/lib/
#   - build wlfulclient.jar: /opt/bea/jdk1.6.0_27/bin/java -jar wljarbuilder.jar
# 
# - Also enabled "enable tunneling" under the protocols section of the managed server from the console.
#

