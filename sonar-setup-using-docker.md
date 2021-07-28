### Sonar setup using docker

Step-1: Create EC2 Ubuntu ami instance with type t2.medium (because, sonar minimum required 4GB RAM)

Step-2: Connect to ubuntu instance and then install docker 'apt update -y && apt install docker.io -y'

Step-3: Install sonar using docker image: 'sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:7.9.5-community'

Step-4: update maven pom.xml file with below snippet

      
