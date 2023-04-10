<h1> Setting up a Maven Project with Jenkins and Nexus </h1>
<h2> This guide provides a step-by-step process for setting up a Maven project with Jenkins and Nexus on Ubuntu Server.</h2>

![Image 1](https://user-images.githubusercontent.com/13016369/230787363-31019a31-35ae-4a06-b54f-2d963c766f39.png)

Prerequisites

   <li> Ubuntu Server
    Docker installed
    OpenJDK 8 installed </li>
    
    
Step 1: Install Docker on the Ubuntu Server
 
      apt install docker.io
      
Step 2: Install Jenkins in the Docker container
 
        docker run -p 8080:8080 -p 50000:50000 -d \
        -v jenkins_home:/var/jenkins_home \
        -v /var/run/docker.sock:/var/run/docker.sock \
         -v $(which docker):/usr/bin/docker \
          jenkins/jenkins:lts   

Step 3:  Retrieve the default Jenkins Password

         docker exec -u 0 it <containter id> bash
         cat /v ar/jenkins_home/secrets/initialAdminPassword
         
Step 4: Launch Jenkins with :8080
        Launch Jenkins with <your-public-IP>:8080
    
Step 5: Install Jenkins plugins
    
    Install the following plugins in Jenkins:

            Maven Integration plugin
            Nexus Artifact Uploader plugin
            Pipeline plugin
            Git plugin
    
Step 6:  Install Nexus on the Ubuntu Server
    
    
    
                    apt install openjdk-8-jre-headless

                    cd /opt/

                    wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz

                    tar -zxvf  latest-unix.tar.gz

                    ls -l

                    chown -R nexus:nexus nexus-3.51.0-01

                    chown -R nexus:nexus sonatype-work

                    vim /opt/nexus-3.51.0-01/bin/nexus.rc 
                    **** (edit the nexus.rc file and specify the username as "nexus".)

                    usermod -aG sudo nexus

                    su - nexus

                    sudo /opt/nexus-3.51.0-01/bin/nexus start

                    ps aux | grep nexus  (This is to check status)

                    netstat -lnpt  (This is to check netstat)
    
 Step 7: Launch the Nexus application using :8081
          
                    <Your-Public-IP>:8081

                        
Step 8: Retrieve the Nexus default password
                       
       Retrieve the default password on your Ubuntu Server using the command:
                        
                        vim /opt/sonatype-work/nexus3/admin.password
Step 9: Step 9: Create a new Maven project in Jenkins

        Create a new Maven project in Jenkins by clicking "New Item" in the Jenkins dashboard and selecting "Maven Project". Give the project a name and click "OK".
                        
Step 10: Configure the Maven project

Configure the Maven project by setting the following properties:

    Under "Source Code Management", select "Git" and enter the repository URL for your Java Maven app hosted on GitHub or another Git hosting service.

    Under "Build", add the following Maven command to build your project: clean install

    Under "Post-build Actions", select "Deploy artifacts to Nexus" and configure the Nexus server by providing the URL, username, and password for your Nexus repository hosted on AWS.
                        
Step 11: Create a new pipeline in Jenkins

       Create a new pipeline in Jenkins by clicking "New Item" in the Jenkins dashboard and selecting "Pipeline". Give the pipeline a name and click "OK".
Step 12: Configure the pipeline

        Configure the pipeline by entering the following script in the "Pipeline Script" box:
                        
                        pipeline {
                                  agent any
                                  stages {
                                      stage('Build') {
                                          steps {
                                              sh 'mvn clean install'
                                          }
                                      }
                                      stage('Deploy') {
                                          steps {
                                              nexusArtifactUploader(
                                                  nexusVersion: 'nexus3',
                                                  protocol: 'http',
                                                  nexusUrl: 'http://<your-nexus-ip>:8081/nexus/repository/<your-repo-name>',
                                                  groupId: 'com.example',
                                                  version: '1.0-SNAPSHOT',
                                                  repository: 'snapshots',
                                                  credentialsId: 'nexus-credentials',
                                                  artifacts: [
                                                      [artifactId: 'your-artifact-id', type: 'jar', file: 'target/your-artifact-id.jar']
                                                  ]
                                              )
                                          }
                                      }
                                  }
}
                        
                     


                

 
