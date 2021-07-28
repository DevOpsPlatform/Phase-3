### Sonar setup using docker

Step-1: Create EC2 Ubuntu ami instance with type t2.medium (because, sonar minimum required 4GB RAM)

Step-2: Connect to ubuntu instance and then install docker 'apt update -y && apt install docker.io -y'

Step-3: Install sonar using docker image: 'sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:7.9.5-community'

Step-4: Check the sonar and open the URL in any browser: http://public-ip-address:9000

Step-5: update maven pom.xml file with below snippet

      Add the below line to <properties> section
      
            <sonar.host.url>http://public-ip-address:9000</sonar.host.url>
            
Step-6: Run the maven command

      mvn clean package org.codehaus.mojo:sonar-maven-plugin:3.7.0.1746:sonar
      
      
Sample project: https://github.com/venkatasykam/DevOpsWebApp/blob/web/pom.xml
            

      
