
---

# PIN1

### Summary
This project sets up a continuous integration (CI) pipeline using Jenkins, SonarQube, and Docker Hub to automate testing and building for a Java project. Jenkins will handle build automation, SonarQube will perform code quality analysis, and Docker Hub will store the generated Docker image.

---

## Installation

### Prerequisites
- **Jenkins**: Install Jenkins for CI/CD automation. Ensure you have the required plugins for SonarQube and Docker installed.
- **SonarQube**: Install and configure SonarQube to assess code quality.
- **Docker Hub**: Have a Docker Hub account ready for hosting your Docker images.

### Installation Steps
Here’s how these installation steps could be organized in the README file, focusing on clarity and step-by-step instructions.

---

### Installation

#### Jenkins Installation

To install Jenkins on a Debian-based system, follow these steps:

1. **Update the package index**:
   ```bash
   sudo apt update
   ```

2. **Install OpenJDK 11**:
   ```bash
   sudo apt install openjdk-11-jdk -y
   ```

3. **Add Jenkins repository key**:
   ```bash
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
     https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   ```

4. **Add Jenkins repository to sources list**:
   ```bash
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```

5. **Update package list and install Jenkins**:
   ```bash
   sudo apt-get update
   sudo apt-get install jenkins -y
   ```

---

#### SonarQube Installation

To install SonarQube with PostgreSQL on a Debian-based system:

1. **Set system configurations**:
   ```bash
   cp /etc/sysctl.conf /root/sysctl.conf_backup
   cat <<EOT> /etc/sysctl.conf
   vm.max_map_count=262144
   fs.file-max=65536
   ulimit -n 65536
   ulimit -u 4096
   EOT
   ```

2. **Configure limits for SonarQube**:
   ```bash
   cp /etc/security/limits.conf /root/sec_limit.conf_backup
   cat <<EOT> /etc/security/limits.conf
   sonarqube   -   nofile   65536
   sonarqube   -   nproc    4096
   EOT
   ```

3. **Install OpenJDK and PostgreSQL**:
   ```bash
   sudo apt-get update -y
   sudo apt-get install openjdk-11-jdk -y
   ```

4. **Install PostgreSQL and set up database for SonarQube**:
   ```bash
   wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
   sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
   sudo apt install postgresql postgresql-contrib -y
   sudo systemctl enable postgresql.service
   sudo systemctl start postgresql.service
   sudo -i -u postgres psql -c "CREATE USER sonar WITH ENCRYPTED PASSWORD 'admin123';"
   sudo -i -u postgres psql -c "CREATE DATABASE sonarqube OWNER sonar;"
   sudo -i -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;"
   ```

5. **Download and Configure SonarQube**:
   ```bash
   sudo mkdir -p /sonarqube/
   cd /sonarqube/
   sudo curl -O https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.3.0.34182.zip
   sudo apt-get install zip -y
   sudo unzip -o sonarqube-8.3.0.34182.zip -d /opt/
   sudo mv /opt/sonarqube-8.3.0.34182/ /opt/sonarqube
   sudo groupadd sonar
   sudo useradd -c "SonarQube - User" -d /opt/sonarqube/ -g sonar sonar
   sudo chown sonar:sonar /opt/sonarqube/ -R
   ```

6. **Edit SonarQube Configuration**:
   ```bash
   cat <<EOT> /opt/sonarqube/conf/sonar.properties
   sonar.jdbc.username=sonar
   sonar.jdbc.password=admin123
   sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
   sonar.web.host=0.0.0.0
   sonar.web.port=9000
   sonar.web.javaAdditionalOpts=-server
   sonar.search.javaOpts=-Xmx512m -Xms512m -XX:+HeapDumpOnOutOfMemoryError
   sonar.log.level=INFO
   sonar.path.logs=logs
   EOT
   ```

7. **Create SonarQube Service**:
   ```bash
   cat <<EOT> /etc/systemd/system/sonarqube.service
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

   LimitNOFILE=65536
   LimitNPROC=4096

   [Install]
   WantedBy=multi-user.target
   EOT

   systemctl daemon-reload
   systemctl enable sonarqube.service
   ```

8. **Install and Configure NGINX**:
   ```bash
   apt-get install nginx -y
   rm -rf /etc/nginx/sites-enabled/default
   rm -rf /etc/nginx/sites-available/default
   cat <<EOT> /etc/nginx/sites-available/sonarqube
   server {
       listen      80;
       server_name sonarqube.example.com;

       access_log  /var/log/nginx/sonar.access.log;
       error_log   /var/log/nginx/sonar.error.log;

       location / {
           proxy_pass  http://127.0.0.1:9000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   EOT

   ln -s /etc/nginx/sites-available/sonarqube /etc/nginx/sites-enabled/sonarqube
   systemctl enable nginx.service
   ```

---

3. **Docker**: Download Docker from [Docker](https://www.docker.com/).

---

## Configuration and Integration

### Jenkins and SonarQube/Docker Hub Integration
1. Install the plugins required in Jenkins:

<img width="659" alt="image" src="https://github.com/user-attachments/assets/3bbca239-dcc7-411e-a1f9-f20c059bd933">
   
2. Generate token in SonarQube
<img width="704" alt="image" src="https://github.com/user-attachments/assets/7041f268-4fe3-4e53-8893-a85bed8480e7">
<img width="327" alt="image" src="https://github.com/user-attachments/assets/2a628443-5550-4e26-a9bb-01e611b09071">

3. Add the token in the Jenkins console

<img width="205" alt="image" src="https://github.com/user-attachments/assets/69727bf4-b481-4f4c-820a-8e87ba0efd2c">

<img width="507" alt="image" src="https://github.com/user-attachments/assets/2ececae5-777d-409c-9d22-96f8bce786f4">
<img width="423" alt="image" src="https://github.com/user-attachments/assets/b030d30f-e71f-46d0-98f2-f56e0272d254">
<img width="585" alt="image" src="https://github.com/user-attachments/assets/47c7acbf-71f1-4d3b-9117-0dacd5aacc95">

4. Add docker hub credentials:

<img width="635" alt="image" src="https://github.com/user-attachments/assets/5c12e3dd-75a2-4626-8f99-bc7115d0ae4f">

5. Install docker engine in the Jeknin server
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
# Install docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
6. Add Docker group to user jenkins and reboot the appliance
```
usermod -aG docker jenkins
reboot
```
7. Add Maven, Java, SonarQube installations

<img width="619" alt="image" src="https://github.com/user-attachments/assets/052b7473-663b-4ee8-a009-0952e44af7c2">
<img width="597" alt="image" src="https://github.com/user-attachments/assets/e96b5f02-bc55-4427-adfc-9e1c365396c1">
<img width="593" alt="image" src="https://github.com/user-attachments/assets/ebbcdd6e-49f7-42bc-b2ad-fdec1885cdca">

8. 
   

---

## Pipeline Workflow

1. **Build Stage**: Jenkins triggers a build for the Java project.
2. **Test Stage**: Jenkins runs tests, capturing test results and generating reports.
3. **SonarQube Analysis**: Jenkins runs SonarQube analysis, checking code quality and reporting issues.
4. **Docker Image Creation**: After tests pass, Jenkins builds a Docker image.
5. **Push to Docker Hub**: The built Docker image is tagged and pushed to Docker Hub.

---
