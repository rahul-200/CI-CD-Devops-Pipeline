# CI/CD Devops Pipeline
## Introduction
In today's fast-paced development environment, continuous integration and continuous delivery (CI/CD) are essential practices for delivering high-quality software rapidly and reliably. In this project, we will walk through setting up a robust CI/CD pipeline from scratch, utilizing industry-standard tools like Jenkins, Docker, Trivy, SonarQube, and Nexus. This guide is designed for developers and DevOps enthusiasts who want to automate the process of building, testing, and deploying applications in a scalable and secure manner.

We will begin by setting up the necessary infrastructure on AWS, followed by the installation and configuration of Docker, Jenkins, Trivy, Nexus, and SonarQube. Finally, we will create a Jenkins pipeline that automates the entire CI/CD process, ensuring your applications are continuously built, scanned for vulnerabilities, analyzed for code quality, and deployed with minimal manual intervention.

## PHASE 1: INFRASTRUCTURE SETUP 🛠️
### 1. Creating 3 Ubuntu 24.04 VM Instances on AWS 🌐
 ### Sign in to the AWS Management Console:

Go to AWS Management Console.

Sign in with your AWS account credentials.

### Navigate to EC2:

Type "EC2" in the search bar or select "Services" > "EC2" under the "Compute" section.

### Launch Instance:

Click "Instances" in the EC2 dashboard sidebar.

Click the "Launch Instance" button.

### Choose an Amazon Machine Image (AMI):

Select "Ubuntu" from the list of available AMIs.

Choose "Ubuntu Server 24.04 LTS".

Click "Select".

### Choose an Instance Type:

Select an instance type (e.g., t2.micro for testing).

Click "Next: Configure Instance Details".

### Configure Instance Details:

Configure optional settings or leave them as default.

Click "Next: Add Storage".

### Add Storage:

Specify the root volume size (default is usually fine).

Click "Next: Add Tags".

### Add Tags:

Optionally, add tags for better organization.

Click "Next: Configure Security Group".

### Configure Security Group:

Allow SSH access (port 22) from your IP address.

Optionally, allow other ports (e.g., HTTP port 80, HTTPS port 443).

Click "Review and Launch".

### Review and Launch:

Review the instance configuration.

Click "Launch".

### Select Key Pair:

Select an existing key pair or create a new one.

Check the acknowledgment box.

Click "Launch Instances".

### Access Your Instance:

Use an SSH client like MobaXterm:

Open MobaXterm and click "Session" > "SSH".

Enter the public IP address of your instance.

Select "Specify username" and enter "ubuntu".

Under "Advanced SSH settings", select "Use private key" and browse to your key pair file (.pem).

Click "OK" to connect.


### 2. Install Docker on All 3 VMs 🐳
### Step-by-Step Installation:

### Install prerequisite packages:
       sudo apt-get update
       sudo apt-get install ca-certificates curl
       
### Download and add Docker's official GPG key:
      sudo install -m 0755 -d /etc/apt/keyrings
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
      sudo chmod a+r /etc/apt/keyrings/docker.asc
      
### Add Docker repository to Apt sources:
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

### Update package index:
     sudo apt-get update

### Install Docker packages:
      sudo apt-get install docker-ce docker-ce-cli containerd.io -y

### Grant permission to Docker socket (optional, for convenience):
      sudo chmod 666 /var/run/docker.sock

By following these steps, you should have successfully installed Docker on your Ubuntu system. You can now start using Docker to containerize and manage your applications.


## Setting Up Jenkins on Ubuntu 🔧
### Step-by-Step Installation:

### Update the system:
    sudo apt-get update
    sudo apt-get upgrade -y

### Install Java (Jenkins requires Java):
    sudo apt install -y fontconfig openjdk-17-jre

### Add Jenkins repository key:
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

### Add Jenkins repository:
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

### Update the package index:
    sudo apt-get update

### Install Jenkins:
    sudo apt-get install -y jenkins

### Start and enable Jenkins:
    sudo systemctl start jenkins
    sudo systemctl enable jenkins

### Access Jenkins:

  Open a web browser and go to http://your_server_ip_or_domain:8080.
  
You will see a page asking for the initial admin password. Retrieve it using:

           sudo cat /var/lib/jenkins/secrets/initialAdminPassword
           
         - Enter the password, install suggested plugins, and create your first admin user.


## Installing Trivy on Jenkins Server 🔍
### Step-by-Step Installation:

### Install prerequisite packages:
     sudo apt-get install wget apt-transport-https gnupg lsb-release
### Add Trivy repository key:
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
### Add Trivy repository to sources:
    echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
### Update package index:
    sudo apt-get update
### Install Trivy:
    sudo apt-get install trivy

## Setting Up Nexus Repository Manager Using Docker 📦
### Step-by-Step Installation:

### Pull the Nexus Docker image:
    sudo docker pull sonatype/nexus3
### Run the Nexus container:
    sudo docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
### Access Nexus:

Open a web browser and go to http://your_server_ip_or_domain:8081.

### The default username is admin. Retrieve the initial admin password from the log:
    sudo docker logs nexus 2>&1 | grep -i password
    - Complete the setup wizard.


## Setting Up SonarQube Using Docker 📈
### Step-by-Step Installation:

### Create a network for SonarQube and PostgreSQL:
    sudo docker network create sonarnet
### Run PostgreSQL container:
    sudo docker run -d --name sonarqube_db --network sonarnet -e 
    POSTGRES_USER=sonar -e POSTGRES_PASSWORD=sonar -e 
    POSTGRES_DB=sonarqube -v postgresql:/var/lib/postgresql -v 
    postgresql_data:/var/lib/postgresql/data postgres:latest
### Run SonarQube container:
    sudo docker run -d --name sonarqube --network sonarnet -p 9000:9000 -e sonar.jdbc.url=jdbc:postgresql://sonarqube_db:5432/sonarqube -e sonar.jdbc.username=sonar -e sonar.jdbc.password=sonar -v sonarqube_data:/opt/sonarqube/data -v sonarqube_extensions:/opt/sonarqube/extensions sonarqube:latest
### Access SonarQube:

Open a web browser and go to http://your_server_ip_or_domain:9000.

The default login is admin with the password admin.

Complete the setup by configuring the SonarQube server.

## Setting Up a Jenkins Pipeline to Automate CI/CD Process 🚀
### Step-by-Step Pipeline Setup:

### Create a New Pipeline Job:

In Jenkins, click on "New Item" and select "Pipeline".

Name the pipeline and click "OK".

Configure the Pipeline:

Scroll down to the "Pipeline" section.

Select "Pipeline script" and define your pipeline stages using Groovy.

### Sample Pipeline Script:
    pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    docker.build("your-app:latest").push("your-docker-repo/your-app:latest")
                }
            }
        }
        stage('Security Scan with Trivy') {
            steps {
                sh 'trivy image your-docker-repo/your-app:latest'
            }
        }
        stage('Quality Analysis with SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube Server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
    }
}
### Save and Run the Pipeline:

Save the pipeline configuration.

Click "Build Now" to run the pipeline.

This pipeline will automate the entire CI/CD process, from cloning the repository to building the application, scanning it for vulnerabilities with Trivy, and analyzing code quality with SonarQube.
