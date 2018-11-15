# JenQube
Implementing Jenkins and SonarQube using Docker Compose to scan a GitHub repo


**Aim**: Implement SonarQube in a docker container and perform a single scan on a Vulnerable GitHub repo
1.	Write a docker compose file to spin up SonarQube and Jenkins
2.	Configure SonarQube Scanner in Jenkins
3.	Create a Jenkins job to run scan a GitHub repo
4.	See results in SonarQube console


**Environment/Service Versions Used**:
Host System – Ubuntu 16.04 LTS in VMware Workstation – 4 GB RAM, 25 GB storage 

Target Github Repository: [DVWA](https://github.com/ethicalhack3r/DVWA)

Jenkins: 2.60.3

SonarQube: 7.1

Postgres: 9.6


**Steps**:
1. Install [Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04) and [Docker Compose](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-16-04) for Ubuntu.

2. Create a docker compose yml file and define services in it
```
version: '2'
services:
  jenkins:
    image: jenkins:latest
    ports:
      - "8080:8080"
      - "50000:50000"
    networks:
      - jenkins
    volumes:
      - jenkins-data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
  postgres:
    image: postgres:9.6
    networks:
      - jenkins
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonarpasswd
    volumes:
      - /var/postgres-data:/var/lib/postgresql/data
  sonarqube:
    image: sonarqube:latest
    ports:
      - "9000:9000"
      - "9092:9092"
    networks:
      - jenkins
    environment:
      SONARQUBE_JDBC_USERNAME: sonar
      SONARQUBE_JDBC_PASSWORD: sonarpasswd
      SONARQUBE_JDBC_URL: "jdbc:postgresql://postgres:5432/sonar"
    depends_on:
      - postgres

networks:
  jenkins:

volumes:
    jenkins-data: {}
```


3. Start docker-compose services
Use the `docker-compose up -d` command to run the containers (root permissions required).
To interact with the containers, run the `docker exec -it <container-id> bash` command. Container IDs can be obtained by running `docker ps`.  

4. Configure SonarQube and Jenkins

Setting up Jenkins:
- Access the Jenkins web interface in a browser on `http://localhost:8080`. To unlock Jenkins an initial admin password is required and can be obtained by viewing the contents of `/var/jenkins_home/secrets/initialAdminPassword` file. 
- This can be done by interacting with the docker image, by running the command: `docker exec -it <jenkins-container-id> bash`. View the initial password using `cat` and enter it to proceed with the setup.
- Continue the setup process by selecting install suggested plugins. Create the first admin user and start Jenkins. Once Jenkins is up and running, go to `Manage Jenkins > Manage Plugins`, and install SonarQube Scanner from the available plugins.

Configuring Jenkins:
- From the Jenkins Dashboard, go to `Manage Jenkins > Configure System`. Make a note of the Jenkins Home Directory mentioned at the top of the page.
- Add a new environment variable under `Global Properties` named `SONAR_USER_HOME`, with its value as the Jenkins home directory path.
 
- Make sure to enable injection of SonarQube server configuration as build environment variables. Add the SonarQube server location in the Server URL field.   
- Select the `Add SonarQube Installation` option, and then add required details (server version 5.3+ will ask for an authentication token, which can be created after accessing the SonarQube server web interface)
 
- Under the `Jenkins Location` sub-section, add the Jenkins host’s location in the `Jenkins URL` field.
SonarQube Scanner Configuration:
- `Manage Jenkins > Global Tools Configuration`
- Select `Add SonarQube Scanner` and select the version to be installed.  

Create and Configure Jenkins Project:
- From the Dashboard, select `New Item`, choose `Freestyle Project` and name it.
- Under `Source Code Management`, select Git and add the target repository that is to be scanned.  
- Prepare the build environment.  
- Under the `Build` section, add a build step to `Execute SonarQube Scanner`. The parameters added under `Analysis Properties` will be passed to the `sonar.properties` file.  


5. Run a build
Once config is complete, go to the project and select `Build Now`. Build progress can be monitored using `Console Output`.  


6. View Results
The scan results can be viewed by either clicking on the SonarQube link under the Jenkins project, or by manually accessing the project on SonarQube web.
