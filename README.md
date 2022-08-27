# Sonarqube Installation and Demo:


- SonarQube is one of the popular static code analysis tools.  
- SonarQube is open-source, Java based tool for Continuous inspection of Code Quality, Code Bugs, Vulnerabilities and Code Duplications.
- It also needs database as well - Database can be MS SQL, Oracle or PostgreSQL. 
- For this demo, We will use PostgreSQL as it is open source as well.
- With the help of SonarQube scanner plugin, you can integrate your jenkins with SonarQube for continuous inspection. 
- Earlier that was Sonar Runner upto version 5.5, now its called sonar scanner and supported with version later than Java 7.
- Alternatives to Sonarqube:
  - Embold
  - Coverity
  - Checkmarx
  - CodeScan
## Pre-requisites for Installing Sonarqube:
  - Instance should have at least 2 GB RAM. For AWS, instance should new and type should be t2.small. For Azure it should be standard B1ms which is 2 GB RAM.
  - Port 9000 should be open.
  - Jenkins should be up and running as we will also integrate jenkins with sonarqube.

![image](https://user-images.githubusercontent.com/92631457/187044358-b8ff5f11-2f71-46fd-a2dc-e96169125afa.png)

## STEP 1 :
   ## Install Java
   - As It is Java based tool, so Java Should be installed as the dependency!
```sh
   apt update
   apt install default-jdk -y
   java --version
```
![image](https://user-images.githubusercontent.com/92631457/187044572-93e7934f-bf4c-4d0d-805d-303312323644.png)

## STEP 2:
  ## Install PostgreSQL:
  
```sh
   # Create the file repository configuration:
   sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
   
   # Import the repository signing key:
   wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   
   
   sudo apt-get update
   
   # Install the latest version of PostgreSQL.
   # If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
   sudo apt-get -y install postgresql
```

![image](https://user-images.githubusercontent.com/92631457/187044770-c1952d50-b9cb-45b0-840d-3e15d98b62d5.png)

   - Start and Enable PostgreSQL:

```sh
   systemctl start postgresql
   systemctl enable postgresql
   systemctl status postgresql
```

![image](https://user-images.githubusercontent.com/92631457/187044880-0a0de9aa-11cb-4ea9-a3f6-833f1a81ead8.png)

![image](https://user-images.githubusercontent.com/92631457/187044905-2f97371c-2053-4a4c-a590-09438c41c973.png)

  - Login as postgresUser:

```sh
   sudo su - postgres
```
  - Now create a user by executing the following command:

```sh
   createuser sonar
```
   
## STEP 3
  ## Switch to SQL shell:
  
```sh
   psql
```

![image](https://user-images.githubusercontent.com/92631457/187045061-56dca493-7f39-4ca9-bfcb-5aeda6f334f7.png)
 
 - Execute the following three lines one by one and type \q to come out of the shell:
 
```sh
   
   ALTER USER sonar WITH ENCRYPTED password 'password';
   
   CREATE DATABASE sonarqube OWNER sonar;
   
   \q
```

![image](https://user-images.githubusercontent.com/92631457/187045178-822663f3-800a-4f0b-8cc1-b407bc3e5c57.png)


 - Type exit to come out of sonar user.
 
## STEP 4:
  ## Now install Sonarqube Webapp:
  
```sh
   sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.7.zip
   
   sudo apt-get -y install unzip
   sudo unzip sonarqube-7.7.zip -d /opt
   
   sudo mv /opt/sonarqube-7.7 /opt/sonarqube -v
```
 - Create Group and User:
 
  ```sh 
     sudo groupadd sonar
  ```
  - Now add the user with directory access
  
  ```sh
     sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
     sudo chown sonar:sonar /opt/sonarqube -R
  ```
  
  - Modify sonar.properties file
  
```sh
   sudo vi /opt/sonarqube/conf/sonar.properties
   #uncomment the below lines by removing # and add values highlighted yellow
   sonar.jdbc.username=sonar
   sonar.jdbc.password=password
   Next, uncomment the below line, removing #
   sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
   #Press escape, and enter :wq! to come out of the above screen.
```
![image](https://user-images.githubusercontent.com/92631457/187045848-c8e99a9a-aec8-43f9-8698-e7e0ffafc068.png)


- Edit the sonar script file and set RUN_AS_USER

```sh
   sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh
   #Add enable the below line 
   RUN_AS_USER=sonar
```
- Create Sonar as a service (this will enable to start automatically when you restart the server)

```sh
   sudo vi /etc/systemd/system/sonar.service
```
- add the below code

```sh
    [Unit]
    Description=SonarQube service
    After=syslog.target network.target

    [Service]
    Type=forking

    ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
    ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

    User=sonar
    Group=sonar
    Restart=always

    [Install]
    WantedBy=multi-user.target
  ```
![image](https://user-images.githubusercontent.com/92631457/187046707-76e9f477-4b07-4fb9-af2a-fa4dcedbd3b5.png)
  
 - Start enable and check the service:
 
```sh
   sudo systemctl start sonar

   sudo systemctl enable sonar

   sudo systemctl status sonar
```
![image](https://user-images.githubusercontent.com/92631457/187046738-58a9b61f-1a69-47ee-b4da-df02c91ecc85.png)

 - Reboot your system and check the sonar logs to check the status:

```sh
   tail -f /opt/sonarqube/logs/sonar.log
 ```
 ![image](https://user-images.githubusercontent.com/92631457/187049068-05806248-abc0-4988-9657-6b8bc1bf4a59.png)

 - Now access sonarQube UI by going to browser and enter public dns name with port 9000
 - Now to go to browser --> http://your_SonarQube_publicdns_name:9000/
 
 ![image](https://user-images.githubusercontent.com/92631457/187049089-3e95ef60-5229-46b4-a0a9-46c41eed2d3b.png)



## STEP 5:
  ## Integrating Sonarqube with jenkins for CI/CD:
  
  - Login to your jenkins and install the sonarqube scanner plugin from manage jenkins
  
  ![image](https://user-images.githubusercontent.com/92631457/187049640-bbc07909-09a7-4639-ba0d-f7720707d978.png)

  - Now login to SonarQube using admin/admin and click on Admin on your top side.
  
  ![image](https://user-images.githubusercontent.com/92631457/187049660-dca5e493-97d8-4b73-a133-f914fde16a9c.png)
  
  - Click on My Account, Security. Under Tokens, Give some value for token name and click on generate Tokens. Copy the token.

  ![image](https://user-images.githubusercontent.com/92631457/187049688-ecd93e6a-06e0-41d4-860e-3bce7733a183.png)
  
  - Now go to dashboard --> Admin --> Credentials --> User --> Add credentials (Unrestricted)
  
  ![image](https://user-images.githubusercontent.com/92631457/187049888-616b0c71-0da8-45f3-aef3-02efd8053304.png)

  
  - Now go to -- Manage Jenkins --> Configure System --> SonarQube installation.
    - Click on Enable injection of Sonarqube server configuration check box.
    - Enter name as SonarQube,
    - URL as http://your_sonarqube_public_dns:9000, no / in the end
    - Paste the token you copied from step #1 by click on Add Credentials, choose Secret Text as dropdown, paste the token as token.
    - Save
    - Click on your existing free style job, click on configure. click on prepare Sonarqube scanner  environment.
    
![image](https://user-images.githubusercontent.com/92631457/187050339-1a898b8e-b6e6-423b-82e3-b07c95f4c773.png)

 - Enter maven goal as clean install sonar:sonar
 
 ![image](https://user-images.githubusercontent.com/92631457/187050074-7be21020-a101-4317-81a3-b910ea7b560c.png)

 - Click on save and build the job.
 - You will see that Jenkins will integrate with SonarQube which does code analysis of your project.
Login to SonarQube, click on Projects to see the project dash board.
 
 ![image](https://user-images.githubusercontent.com/92631457/187051002-7d656f4a-288f-472e-98e3-07bcddce5533.png)

 - You can also make a pipeline job and let sonarqube do it job of continuous inspection.
 


  



   
   
   
   
   
   
   
   
   
   
