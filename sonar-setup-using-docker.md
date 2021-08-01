### Sonar setup using docker

Step-1: Create EC2 Ubuntu ami instance with type t2.medium (because, sonar minimum required 4GB RAM)

Step-2: Connect to ubuntu instance and then install docker 'apt update -y && apt install docker.io -y'

Step-3: Install sonar using docker image: 'sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:7.9.5-community'

Step-4: Check the sonar and open the URL in any browser: http://public-ip-address:9000

Step-5: update maven pom.xml file with below snippet

      Add the below line to <properties> section
      
            <sonar.host.url>http://public-ip-address:9000</sonar.host.url>
            
Step-6: Run the maven command

      mvn clean package org.codehaus.mojo:sonar-maven-plugin:3.7.0.1746:sonar -DreleaseVersion=1.0
      
      or - without updating pom.xml file
      
      mvn clean install org.codehaus.mojo:sonar-maven-plugin:3.7.0.1746:sonar -Dsonar.projectKey=groupId:artifactId -Dsonar.host.url=http://localhost:9000 -Dsonar.login=loginHASH -Dsonar.sources=src/main/java/ -Dsonar.java.binaries=target/classes
      
      
Sample project: https://github.com/venkatasykam/DevOpsWebApp/blob/web/pom.xml


#### Maven installation on ubuntu

      apt update -y && apt upgrade -y && apt install openjdk-8-jdk -y
      
      java -version
      
      find / -name "tools.jar"
      
      export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
      
      sudo apt install maven -y
      
      mvn -v

#### Sonar with Jenkins

Step-1: Install Jenkins using docker

      sudo docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:latest
      
Step-2: Install sonarqube scanner plugin

      Manage Jenkins >> Manage Plugin >> Available - search for 'scanner' >> select sonarquve scanner plugin >> install without restart
      
![image](https://user-images.githubusercontent.com/24622526/127777873-e6d69b78-eba6-4fea-92b5-d8cdb51f9e99.png)

Step-3: Configure Sonar in Jenkins

      Manage Jenkins >> Configure System >> SonarQube servers
      
![image](https://user-images.githubusercontent.com/24622526/127777517-aa234189-a9f4-435c-91e3-372eb60f5197.png)

Step-4: Install maven on Jenkins

      Maven Jenkins >> Global Tool Configuration >> Maven >> Add Maven 
      
      Name: maven-3.8.1
      
![image](https://user-images.githubusercontent.com/24622526/127777598-09cc47ac-6183-4402-8260-10c310cda7ca.png)

Step-5: Configure job in jenkins

      New Item >> Pipeline >> copy and paste the below snippet in Job configuration
      
---

      pipeline {
          agent any

            tools {
                        maven "maven-3.8.1"
            }

          stages {
              stage('Clone sources') {
                  steps {
                      git url: 'https://github.com/venkatasykam/DevOpsWebApp.git'
                  }
              }

              stage('SonarQube analysis') {
                  steps {
                      withSonarQubeEnv('SonarQube') {
                          sh "mvn clean package org.codehaus.mojo:sonar-maven-plugin:3.7.0.1746:sonar -DreleaseVersion=1.0"
                      }
                  }
              }
              stage("Quality gate") {
                  steps {
                      waitForQualityGate abortPipeline: true
                  }
              }
          }
      }

Step-6: Run the Jenkins job  
