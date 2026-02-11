# open_telemetry_microservices
# Micro Services Project

## OpenTelemetry Astronomy Shop Demo

This project contains the Open Telemetry Astronomy Shop, a microservice-based distributed system intended to illustrate the implementation of Open Telemetry in a near real-world environment.

### Project Goals
- Provide a realistic example of a distributed system that can be used to demonstrate OpenTelemetry instrumentation and observability.
- Build a base for vendors, tooling authors, and others to extend and demonstrate their OpenTelemetry integrations.
- Create a living example for OpenTelemetry contributors to use for testing new versions of the API, SDK, and other components or enhancements.

### Supported Languages
Open Telemetry supports many popular programming languages including:
- Python
- Java
- JavaScript / Node.js
- Go
- .NET
- Ruby
- PHP

## Micro Services Architecture

<img width="1166" height="1044" alt="image" src="https://github.com/user-attachments/assets/9ce5f0d0-c669-4a5a-a941-07c3937bed42" />

This architecture illustrates a microservices-based e-commerce architecture using various communication protocols (mainly gRPC, HTTP, and TCP), and integrates tools like Kafka, Envoy, Valkey, and Flagd for key functionalities.

### Top Layer – Entry Points & Proxy
- Internet, Load Generator, React Native App
  - These represent external users or automated testing tools accessing the application via HTTP.
- Frontend Proxy (Envoy)
  - Acts as an API Gateway or Ingress Controller.
  - Routes HTTP traffic to internal services like Frontend, Image Provider, and flagd-ui.
  - Offers load balancing, routing, and security features.

### Frontend Layer
- Frontend
  - Serves the web UI and connects users with backend services.
  - Communicates via gRPC to:
    - Ad, Cart, Checkout, Currency, Recommendation, Product Catalog.
- flagd-ui
  - UI dashboard for feature flag management (likely connects to Flagd).
- Image Provider (nginx)
  - Serves static assets (e.g., product images).

### Core Services
- Checkout
  - Central service in the buying workflow.
  - Communicates with:
    - Shipping, Quote, Email, Currency, Payment, Fraud Detection, Flagd.
- Cart
  - Maintains user carts, backed by Cache (Valkey) and integrated with:
    - Ad, Flagd.
- Ad
  - Provides ad content, possibly for recommendations or marketing.
- Recommendation
  - Suggests products to users based on various data points.
  - Talks to Product Catalog.
- Product Catalog
  - Stores product details, connects to both Recommendation and Frontend.

### Supporting Services
- Cache (Valkey)
  - Used for caching (probably Redis-compatible like Valkey).
  - Supports Cart and Ad.
- queue (Kafka)
  - Event streaming platform.
  - Accepts messages from Checkout, passes them to:
    - Accounting
    - Fraud Detection

### Backend Services
- Accounting
  - Processes financial data from Kafka.
- Fraud Detection
  - Analyzes data from Kafka to detect fraudulent activity.
  - Also talks to Flagd.
- Shipping
  - Handles shipping details; communicates over HTTP and gRPC.
- Quote
  - Generates shipping or price quotes.
- Email
  - Sends confirmation and update emails.
- Currency
  - Converts prices to different currencies.
- Payment
  - Processes transactions, talks to Flagd.

### Feature Flagging
- Flagd
  - Central service managing feature flags.
  - Communicated with by:
    - Cart, Checkout, Fraud Detection, Payment.

### Communication Types
- gRPC – Used extensively for internal service-to-service calls (fast & efficient).
- HTTP – Used for user-facing and REST-based services.
- TCP – Used where event-streaming or lower-level connections are needed (Kafka, service queues).

### Architecture Summary
- Microservices based – each component handles a specific responsibility.
- Event-driven – Kafka is used for decoupling & async processing.
- Service Mesh / Proxy – Envoy acts as the API gateway.
- Caching – via Valkey (Redis-like).
- Observability & Feature Flags – via Flagd and possibly frontend telemetry.
- Platform-Agnostic Access – UI via browser + mobile app (React Native).

### Benefits
- Unified approach to collecting metrics, logs, and traces.
- Vendor-neutral and widely adopted.
- Integrates easily with tools like Prometheus, Grafana, and Jaeger.
- Supports both manual and automatic instrumentation.


## 1. Provisioning EC2 Instance

### Step 1: Launch EC2
- Choose Ubuntu 22.04 LTS
- Instance type: t2.xlarge is a chargeable instance type. Be cautious — make sure to stop or resize it to t2.micro after your practice session to avoid incurring unnecessary charges.
- Open ports in security group:
  - SSH: 22
  - HTTP: 80
  - Jenkins: 8080
  - All port

### Step 2: Connect to Jenkins EC2 server
```bash
ssh -i your-key.pem ubuntu@your-ec2-ip
```

# Jenkins Installation on Ubuntu
```
sudo apt update -y
sudo apt upgrade -y

sudo apt install -y openjdk-17-jdk curl gnupg

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | \
sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg

echo "deb [signed-by=/usr/share/keyrings/jenkins.gpg] https://pkg.jenkins.io/debian-stable binary/" | \
sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update -y

sudo apt install -y jenkins


sudo systemctl enable jenkins
sudo systemctl start jenkins

sudo systemctl status jenkins --no-pager

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
### Access Jenkins UI
Go to: http://<your-ec2-ip>:8080

##  Install Jenkins Plugins
Go to Manage Jenkins → Manage Plugins → Install the following plugins:
- Docker Pipeline
- Kubernetes
- Pipeline
- Blue Ocean (optional)
- Docker
- Kubernetes cli
- Kubernetes credentials

# Docker insatallation on Ubuntu
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo chmod 666 /var/run/docker.sock
sudo usermod -aG docker jenkins

```
##  Configure Docker Credentials in Jenkins

### Step1: Store Docker Credentials in Jenkins
1. Go to Jenkins Dashboard → Manage Jenkins → Credentials
2. Select (global) domain
3. Click Add Credentials
   - Kind: Username with password
   - Username: DockerHub username
   - Password: password
   - ID: docker-cred
   - Description: Docker Hub Credentials

##  Create Multibranch Pipeline

### Step 1: Create a New Multibranch Pipeline Job
- Go to Jenkins Dashboard → New Item
- Name: my-multibranch-job
- Type: Multibranch Pipeline

### Step 2: Configure Branch Source
- Choose GitHub
- Add repository URL: 
```
https://github.com/jadalaramani/open_telemetry_microservices.git
```
- Add GitHub credentials (if private)
- Jenkins will scan all branches and create jobs for each branch with a Jenkinsfile

##  Example Jenkinsfile (for Docker builds)
Create a Jenkinsfile in your GitHub repo root:
```groovy
pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t ramanijadala/frontend:latest ."
                    }
                }
            }
        }      
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push ramanijadala/frontend:latest "
                    }
                }
            }
        }    
    }
}
```

# Kubernetes setup

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
