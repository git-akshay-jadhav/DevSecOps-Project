Embracing DevSecOps: Building a Secure Netflix Clone on AWS with Jenkins, Docker, and Argo CD

Akshay Jadhav's photo
Akshay Jadhav
·
Jan 8, 2024
·
7 min read

In this tutorial, we will walk through the process of deploying a Netflix Clone application on the AWS cloud using Jenkins. This project follows a DevSecOps approach, incorporating security measures at every stage of the development and deployment pipeline.

Phase 1: Initial Setup and Deployment
Step 1: Launch EC2 (Ubuntu 22.04)
Start by provisioning an EC2 instance on AWS with Ubuntu 22.04. Connect to the instance using SSH.


COPY
# Example SSH connection
ssh -i your-key.pem ubuntu@your-ec2-public-ip


Step 2: Clone the Code
Update packages and clone the project code repository.


COPY
# Update packages
sudo apt-get update

# Clone the repository
git clone https://github.com/git-akshay-jadhav/DevSecOps-Project.git
Step 3: Install Docker and Run the App Using a Container
Set up Docker on the EC2 instance and build/run the application.


COPY
# Install Docker
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock

# Build and run the application
docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest
Step 4: Get the API Key
Obtain an API key from TMDB (The Movie Database) for the application.

Open a web browser and navigate to TMDB (The Movie Database) website.

Click on "Login" and create an account.

Once logged in, go to your profile and select "Settings."

Click on "API" from the left-side panel.

Create a new API key by clicking "Create" and accepting the terms and conditions.

Provide the required basic details and click "Submit."

You will receive your TMDB API key.

Now recreate the Docker image with your api key:


COPY
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
Recreate the Docker image with your API key.


COPY
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
Phase 2: Security
Install SonarQube and Trivy
Install SonarQube for code analysis and Trivy for image scanning.


COPY
# Install SonarQube
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

# Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
Integrate SonarQube with CI/CD pipeline and configure it to analyze code.



Phase 3: CI/CD Setup
Install Jenkins for Automation
Install Jenkins on the EC2 instance.


COPY
# Install Java
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version

# Install Jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
Access Jenkins in a web browser using the public IP of your EC2 instance.


COPY
http://your-ec2-public-ip:8080


Install Necessary Plugins in Jenkins
In the Jenkins dashboard, navigate to "Manage Jenkins" → "Manage Plugins" → "Available Plugins" and install the following plugins:

Eclipse Temurin Installer

SonarQube Scanner

NodeJs Plugin

Email Extension Plugin

Configure Java and Node.js in Global Tool Configuration.

Integrate SonarQube and Configure
Create a Jenkins webhook and configure CI/CD pipeline in Jenkins. Use the provided Jenkinsfile for the pipeline.

Phase 4: Monitoring
Install Prometheus and Grafana
Set up Prometheus and Grafana to monitor the application.

Installing Prometheus
Create a dedicated user, download Prometheus, configure, and start.


COPY
# Prometheus setup (replace placeholders)
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
sudo nano /etc/systemd/system/prometheus.service
Paste the Prometheus service configuration, enable, and start.


COPY
sudo systemctl enable prometheus
sudo systemctl start prometheus


Installing Node Exporter
Create a system user, download Node Exporter, configure, and start.


COPY
# Node Exporter setup (replace placeholders)
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
sudo nano /etc/systemd/system/node_exporter.service
Paste the Node Exporter service configuration, enable, and start.


COPY
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
Configure Prometheus to scrape metrics from Node Exporter.

Prometheus Plugin Integration
Integrate Jenkins with Prometheus to monitor the CI/CD pipeline. Configure Prometheus to scrape metrics from Node Exporter and Jenkins.

Installing Grafana
Install Grafana, enable, and start.


COPY
grafana setup
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get -y install grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server


Access Grafana web interface and configure Prometheus as a data source.


COPY
http://your-ec2-public-ip:3000
Phase 5: Extended Pipeline for Docker Image Scanning and Deployment
Below is an extended Jenkins pipeline that includes Docker image scanning using Trivy and deployment to a container.


COPY
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/git-akshay-jadhav/DevSecOps-Project.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=13005dd60a778914ace1ead5ee5a04c0 -t netflix ."
                        sh "docker tag netflix akshay1669/netflix:latest "
                        sh "docker push akshay1669/netflix:latest "
                    }
                }
            }
        }
        stage('TRIVY') {
            steps {
                sh "trivy image akshay1669/netflix:latest > trivyimage.txt"
            }
        }
        stage('Deploy to container') {
            steps {
                sh 'docker run -d -p 8081:80 akshay1669/netflix:latest'
            }
        }
    }
}

post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'Yourmail@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    }
}


This extended pipeline ensures that the Docker image is scanned for vulnerabilities using Trivy before being deployed to a container. Additionally, it sends an email with build details and scan reports after each execution.



Phase 6: Kubernetes Setup and Monitoring
Create Kubernetes Cluster with Nodegroups
In this phase, we'll set up a Kubernetes cluster with node groups, providing a scalable environment for deploying and managing applications.

Create a Kubernetes Cluster:

Set up an Amazon EKS cluster with node groups using the AWS Management Console or AWS CLI. Follow the official documentation for detailed instructions.

Monitor Kubernetes with Prometheus:

Prometheus is a robust monitoring and alerting toolkit. We'll use it to monitor the Kubernetes cluster.

Install Node Exporter using Helm:

Helm is a package manager for Kubernetes that simplifies deployment. Install Node Exporter to collect system-level metrics from cluster nodes.


COPY
 # Add Prometheus Community Helm repository
 helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

 # Create a Kubernetes namespace for the Node Exporter
 kubectl create namespace prometheus-node-exporter

 # Install Node Exporter using Helm
 helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
Configure Prometheus to Scrape Metrics:

Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. Modify your prometheus.yml as follows:


COPY
 - job_name: 'Netflix'
   metrics_path: '/metrics'
   static_configs:
     - targets: ['node1Ip:9100']
Replace 'Netflix' with your desired job name and adjust the target IP accordingly.

Reload or restart Prometheus to apply these changes.

Phase 7: Application Deployment with ArgoCD
Deploy Application with ArgoCD
Now, let's deploy your application using ArgoCD.

Install ArgoCD:

Follow the instructions in the EKS Workshop documentation to install ArgoCD on your Kubernetes cluster.

Set GitHub Repository as a Source:

After installation, configure your GitHub repository as the source for your application deployment. This involves connecting to your repository and defining the source for the ArgoCD application.

Create ArgoCD Application:

Use ArgoCD's declarative syntax in a YAML file to define your application. Here's a basic example:


COPY
 apiVersion: argoproj.io/v1alpha1
 kind: Application
 metadata:
   name: your-app
 spec:
   destination:
     name: your-destination
     namespace: your-namespace
   project: your-project
   source:
     repoURL: https://github.com/your/repo
     path: /path/to/app
     targetRevision: HEAD
   syncPolicy:
     automated: true
     prune: true
     selfHeal: true
Customize the values based on your setup.

Access Your Application:

Ensure that port 30007 is open in your security group. Open a new tab and enter NodeIP:30007 to access your running application.

Phase 8: Cleanup
Cleanup AWS EC2 Instances
Terminate AWS EC2 instances that are no longer needed. Use the AWS Management Console or AWS CLI to terminate instances launched during this project.
