### Sonar setup using docker

Step-1: Create EC2 Ubuntu ami instance with type t2.medium (because, sonar minimum required 4GB RAM)

Step-2: Connect to ubuntu instance and then install docker 'apt update -y && apt install docker.io -y'

Step-3: Install sonar using docker image: 'sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:7.9.5-community'

Step-4: Check the sonar and open the URL in any browser: http://public-ip-address:9000

      Default username: admin
      Default password: admin

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

      Default location of docker volumes: /var/lib/docker/volumes
      
      docker run -d -p 9999:8080 --name jenkins -v jenkins:/var/jenkins_home jenkins/jenkins:latest
      
Step-2: Install sonarqube scanner plugin

      Manage Jenkins >> Manage Plugin >> Available - search for 'sonarqube scanner' >> select sonarqube scanner plugin >> install without restart
      
![image](https://user-images.githubusercontent.com/24622526/127777873-e6d69b78-eba6-4fea-92b5-d8cdb51f9e99.png)

Step-3: Configure Sonar in Jenkins

      (Optional) Generate api token from sonar admin user. Login to Sonar > User Icon > My Account > Security >  give any name ex: jenkinssonar  > Generate > copy the token 

 ![image](https://user-images.githubusercontent.com/24622526/127779787-7b5e4217-5c9e-40e9-a5ff-ed91db532a28.png)

add the token to Jenkins > Manage Jenkins > Manage Credentials > Jenkins > Global Credentials > Add Credentials > 

![image](https://user-images.githubusercontent.com/24622526/127779880-de9b9b0f-828d-4b80-af0c-eda89014c754.png)
 
      Manage Jenkins >> Configure System >> SonarQube servers 
      
> Note: Don't give slash '/' at end of the URL, ex: wrong url: http://sonar-public-ip-address:9000/ and correct url: http://sonar-public-ip-address:9000


![image](https://user-images.githubusercontent.com/24622526/127779758-99981b16-d7e7-4eaf-bc5a-16f2395e84df.png)
      

Step-4: create webhook: http://jenkins-public-ip-address:9999/sonarqube-webhook/

      Login to sonar with admin user: Administration > Configuration > Webhooks > Create
      
![image](https://user-images.githubusercontent.com/24622526/127778846-b797733d-198a-42b4-8974-928477347770.png)

![image](https://user-images.githubusercontent.com/24622526/127779987-f5771d3d-f609-4b4c-a471-a78290e7aecd.png)


Step-5: Install maven on Jenkins

      Maven Jenkins >> Global Tool Configuration >> Maven >> Add Maven 
      
      Name: maven-3.8.1
      
![image](https://user-images.githubusercontent.com/24622526/127777598-09cc47ac-6183-4402-8260-10c310cda7ca.png)

Step-6: Configure job in jenkins

      New Item >> Pipeline >> copy and paste the below snippet in Job configuration


![image](https://user-images.githubusercontent.com/24622526/127778001-20dcb45d-8787-4739-abcd-338946b87cff.png)

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

Step-7: Run the Jenkins job with default sonar quality gates

![image](https://user-images.githubusercontent.com/24622526/127778246-82cc7d9e-99d9-44fc-8a77-c951d0022ad3.png)

![image](https://user-images.githubusercontent.com/24622526/127778551-2b30626e-e2a4-4800-8b36-cc1edb357065.png)

![image](https://user-images.githubusercontent.com/24622526/127779600-4535673b-dcbc-46c4-8293-447a71fd0f34.png)


Create our own quality gates. Sonar >> Quality Gates >> Create


![image](https://user-images.githubusercontent.com/24622526/127778354-154c1f28-b21a-40eb-9864-4229f68104c3.png)

![image](https://user-images.githubusercontent.com/24622526/127778377-e93601a5-6d7c-475a-a54b-f52ea131419b.png)


Add some rules by click on "Add Condition" 

![image](https://user-images.githubusercontent.com/24622526/127778410-b2e8f367-fd4e-4c31-b67a-10522f80794e.png)

![image](https://user-images.githubusercontent.com/24622526/127778439-6659aecb-bb9e-4176-b8ef-6068caa605ee.png)


![image](https://user-images.githubusercontent.com/24622526/127778479-aebff7af-365b-4b36-a8f7-5a78f2e95f90.png)

Update it as "Default" one

![image](https://user-images.githubusercontent.com/24622526/127778484-f204d46e-8bc7-448b-8dd3-de45619e7093.png)

![image](https://user-images.githubusercontent.com/24622526/127778633-d33bce62-2af2-4e02-85f4-91a03c02072c.png)


![image](https://user-images.githubusercontent.com/24622526/127779461-c9666cfb-b4f9-4c92-8bc1-0ebfa76b3c23.png)

