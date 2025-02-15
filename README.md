<div align="center">
  <img src="./public/assets/output.png" alt="Logo" width="100%" height="100%">
</div>

# Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project!

### **Phase 1: Initial Setup and Deployment**

**Step 1: Launch EC2 (Ubuntu 22.04):**

- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.

**Step 2: Clone the Code:**

- Update all the packages and then clone the code.
- Clone your application's code repository onto the EC2 instance:
    
    ```bash
    git clone https://github.com/RajalakshmiNagendran/Devsecops-end-to-end.git
    ```
    

**Step 3: Install Docker and Run the App Using a Container:**

- Set up Docker on the EC2 instance:
    
    ```bash
    
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    ```
    
- Build and run your application using Docker containers:
    
    ```bash
    docker build -t netflix .
    docker run -d --name netflix -p 8081:80 netflix:latest
    
    #to delete
    docker stop <containerid>
    docker rmi -f netflix
    ```

It will show an error cause you need API key

**Step 4: Get the API Key:**

- Open a web browser and navigate to TMDB (The Movie Database) website.
- Click on "Login" and create an account.
- Once logged in, go to your profile and select "Settings."
- Click on "API" from the left-side panel.
- Create a new API key by clicking "Create" and accepting the terms and conditions.
- Provide the required basic details and click "Submit."
- You will receive your TMDB API key.

Now recreate the Docker image with your api key:
```
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
```

### **Phase 2: Security**

1. **Install SonarQube and Trivy:**
    - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
        
        sonarqube
        ```
        docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
        ```
        
        
        To access: 
        
        publicIP:9000 (by default username & password is admin)
        
        To install Trivy:
        ```
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy        
        ```
        
        to scan image using trivy
        ```
        trivy image <imageid>
        ```
        
        
2. **Integrate SonarQube and Configure:**
    - Integrate SonarQube with your CI/CD pipeline.
    - Configure SonarQube to analyze code for quality and security issues.

## **Phase 3: CI/CD Setup**

1. **Install Jenkins for Automation:**
    - Install Jenkins on the EC2 instance to automate deployment:
    Install Java
    
    ```bash
    sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version
    openjdk version "17.0.8" 2023-07-18
    OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
    OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
    
    #jenkins
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    ```
    
    - Access Jenkins in a web browser using the public IP of your EC2 instance.
        
        publicIp:8080
        
2. **Install Necessary Plugins in Jenkins:**

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 Eclipse Temurin Installer (Install without restart)

2 SonarQube Scanner (Install without restart)

3 NodeJs Plugin (Install Without restart)

4 Email Extension Plugin

### **Configure Java and Nodejs in Global Tool Configuration**

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save


### SonarQube

Create the token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

After adding sonar token

Click on Apply and Save

**The Configure System option** is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

Create a Jenkins webhook

1. **Configure CI/CD Pipeline in Jenkins:**
- Create a CI/CD pipeline in Jenkins to automate your application deployment.

```groovy
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
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/RajalakshmiNagendran/Devsecops-end-to-end.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
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
    }
}
```

Certainly, here are the instructions without step numbers:

**Install Docker Tools and Docker Plugins:**

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "Docker."
- Check the following Docker-related plugins:
  - Docker
  - Docker Commons
  - Docker Pipeline
  - Docker API
  - docker-build-step
- Click on the "Install without restart" button to install these plugins.

**Add DockerHub Credentials:**

- To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
  - Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
  - Click on "System" and then "Global credentials (unrestricted)."
  - Click on "Add Credentials" on the left side.
  - Choose "Secret text" as the kind of credentials.
  - Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
  - Click "OK" to save your DockerHub credentials.

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.

```groovy

pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/RajalakshmiNagendran/Devsecops-end-to-end.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
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
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourapikey> -t netflix ."
                       sh "docker tag netflix 9841/netflix:latest "
                       sh "docker push 9841/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image 9841/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 9841/netflix:latest'
            }
        }
    }
}


If you get docker login failed errorr

sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins


```   

### **Phase 4: Notification**

1. **Implement Notification Services:**
    - Set up email notifications in Jenkins or other notification mechanisms.

### **Phase 5: Kubernetes**

## Create Kubernetes Cluster with Nodegroups

In this phase, you'll set up a Kubernetes cluster with node groups. This will provide a scalable environment to deploy and manage your applications.

### Setup ALB Ingress Controller:

1. Go to folder "aws-load-balancer-controller-on-eks-cluster" and execute all commands below.
2. Creating an EKS Cluster
➜ eksctl create cluster -f ./cluster.yaml

3. Install the AWS Load Balancer Controller
2.1 Add the EKS chart repo to helm
➜ helm repo add eks https://aws.github.io/eks-charts

4. Install the AWS Load Balancer Controller CRDs - Ingress Class Params and Target Group Bindings
➜  git clone https://github.com/aws/eks-charts.git , cd eks-charts
➜  kubectl apply -f /stable/aws-load-balancer-controller/crds/

6. Install the helm chart by passing the serviceAccount.create=false adn serviceAccount.name=aws-load-balancer-controller
➜ helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=app-lb-demo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer

7. Deploy an app
➜ kubectl apply -f ./SampleApp.yaml

8. Verify the ingress is created and ALB is provisioned
➜  kubectl get ingress -A
➜  kubectl describe ingress ingress-2048 -n game-2048
<div align="center">
  <img src="./public/assets/Capture2.PNG" alt="Logo" width="100%" height="100%">
</div>

### Install metrics server using Helm and apply Horizontal Pod Autoscaler (HPA):

1. Deploy Metrics Server in Kubernetes using Helm

```bash
  kubectl config set-context $(kubectl config current-context) --namespace=default
  helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server
  helm repo update
  helm show values metrics-server/metrics-server > ~/metrics-server.values
  helm install metrics-server metrics-server/metrics-server --values ~/metrics-server.values
  Once created. verify with the command: helm ls 
  kubectl get all -n metrics-server
  helm upgrade metrics-server metrics-server/metrics-server --values ~/metrics-server.values
  View the node’s resources: kubectl top nodes  
  You can also view the pod resources: kubectl top pods -A
  Configure Horizontal Pod Autoscaling: kubectl apply -f ./SampleApp.yaml & kubectl apply -f hpa.yml
  Using the kubectl autoscale command: kubectl autoscale deployment deployment-2048 --cpu-percent=50 --min=1 --max=3
  Now verify the creation: kubectl get hpa & kubectl describe hpa my-app-hpa
```
<div align="center">
  <img src="./public/assets/Untitled4.png" alt="Logo" width="100%" height="100%">
</div>

### **Phase 6: Monitoring**

* To check the dashboard go to: https://us5.datadoghq.com/dash/integration/423/kubernetes-cluster-overview-dashboard?fromUser=false&refresh_mode=sliding&from_ts=1719749972198&to_ts=1719751772198&live=true
* Kubernetes overview: https://us5.datadoghq.com/orchestration/explorer/pod?explorer-na-groups=false&panel_tab=yaml

1. **Install Datadog agent:**

   Set up Datadog to monitor your application.

<div align="center">
  <img src="./public/assets/dashboard-datadog.png" alt="Logo" width="100%" height="100%">
</div>

   1. Go to https://www.datadoghq.com/ create a login
   2. In Integrations -> Agent -> select kubernetes, follow the steps provided
   3. Follow this link ... https://www.datadoghq.com/blog/monitoring-kubernetes-with-datadog/
   4. Deploy the "cluster-agent-rbac.yaml" and rbac.yaml files
      ```bash
      kubectl create -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/cluster-agent/cluster-agent-rbac.yaml"
      kubectl create -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/cluster-agent/rbac.yaml"
      ```
   5. Now, let’s install Datadog Agent in Kubernetes using Operator. First, if you haven’t installed Datadog helm repository in your cluster already, you can use the command below to install the       
      repository.
      ```bash
      helm repo add datadog https://helm.datadoghq.com
      ```
   6. Then you can use this command to install Datadog Operator in your cluster.
      ```bash
      helm install datadog-operator datadog/datadog-operator
      ```      
<div align="center">
  <img src="./public/assets/install-datadog.png" alt="Logo" width="100%" height="100%">
</div>

   7. After successfully installing Datadog Operator, you should create Secret object that includes your api-key and app-key.
      ```bash
      kubectl create secret generic datadog-secret --from-literal api-key=<DATADOG_API_KEY> --from-literal app-key=<DATADOG_APP_KEY>
      ```
   8. Deploy the "datadog-agent.yaml" files
      ```bash
      kubectl apply -f datadog-agent.yaml
      ```
   9. You can do
      ```bash
      kubectl get datadogagent
      ```
  10. To monitor the agent installation process. After a while, you can confirm on your Datadog Kubernetes installation page that the installation has been successfully completed.
<div align="center">
  <img src="./public/assets/agent-installation-datadog.png" alt="Logo" width="100%" height="100%">
</div>
  11. Finally, when you go to Infrastructure then click Kubernetes, you can now monitor your Kubernetes cluster.
<div align="center">
  <img src="./public/assets/k8s.png" alt="Logo" width="100%" height="100%">
</div>

  **Create alerts in Datadog for critical metrics:**
  1. From the dashboard metrics, click on any of the metric you want to notify, click create metric, give the values according to the snap and click on          create.
<div align="center">
  <img src="./public/assets/Capture.PNG" alt="Logo" width="100%" height="100%">
</div>
