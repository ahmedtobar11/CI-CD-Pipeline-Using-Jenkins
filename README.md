# Automated Docker EKS Deployment Pipeline

This project showcases a complete CI/CD pipeline that integrates Jenkins, Docker, SonarQube, Trivy, and Amazon EKS to automate code fetching, analysis, building, scanning, and deployment of a web application. The pipeline ensures that the application is deployed on Amazon EKS and accessible via a classic load balancer.

## 🗺️​Project Structure
![Animation](https://github.com/user-attachments/assets/6a780496-41f3-4d3b-bed1-26d1616b1c75)

##  📈Project Workflow

1.  **Set Up Jenkins Container**
2.  **Install Tools Inside Jenkins Container**
3.  **Configure Jenkins Plugins and Credentials**
4.  **Set Up SonarQube**
5.  **Create Amazon EKS Cluster**
6.  **Pipeline Stages Overview**
7.  **Pipeline Success Verification**

## 📥Set Up Jenkins Container

Run the following command to create and start a Jenkins container:

```
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
  ```

### Accessing the Jenkins Container

To access the Jenkins container:
```
docker exec -it jenkins bash
```

## ⚒️ Install Tools Inside Jenkins Container

Install the required tools in the Jenkins container to ensure the proper functioning of the pipeline.

### 1. Install Trivy

```
apt-get update && apt-get install -y curl
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
trivy --version
```

### 2. Install AWS CLI
```
apt-get install -y curl unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install
rm -rf awscliv2.zip aws
```

### 3. Install Docker CLI
```
apt-get install -y docker.io
```

### 4. Install `kubectl`
```
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```
### 5. Configure AWS CLI
```
apt install nano less
aws configure
```

Provide the following details:

-   **AWS Access Key ID**
-   **AWS Secret Access Key**
-   **Default region name** 
-   **Default output format** 

## 🔰 Configure Jenkins Plugins and Credentials

### Required Jenkins Plugins

-   **Eclipse Temurin installer**
-   **Maven Integration**
-   **Pipeline Maven Integration**
-   **SonarQube Scanner**
-   **SonarQube Generic Coverage**

### Credentials Needed

-   **SonarQube Token**: Create and add in **Manage Jenkins > Credentials** as **Secret Text**.
-   **Docker Hub Credentials**: Add Docker Hub username and password in **Manage Jenkins > Credentials** as **Username with Password**.
-   **AWS Credentials**: Add AWS Access Key ID and Secret Access Key in **Manage Jenkins > Credentials** as **Secret Text**.
  ![4](https://github.com/user-attachments/assets/044a5479-5c27-4eee-856b-daa1ac54aa8b)


## 🧪 Set Up SonarQube

Run SonarQube locally using Docker:
```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
Configure the SonarQube server in Jenkins under **Manage Jenkins > Configure System**.
![1](https://github.com/user-attachments/assets/c4172694-0ca2-4ed1-bdcf-e7ad895a4747)

## ☁️Create Amazon EKS Cluster

Create an EKS cluster and node group using the AWS Management Console or AWS CLI.
![2](https://github.com/user-attachments/assets/dc75fd34-0d0f-46fb-9989-0c3ea51943bb)


### Update the `kubeconfig`:
```
aws eks --region us-east-1 update-kubeconfig --name EKS-deploy-app
```


## 🎉Verifying Pipeline Success

Ensure all stages complete successfully:

-   **SCM Checkout**: Code is fetched from the repository.
-   **SonarQube Analysis**: Quality and code coverage reports are generated.
-   **Trivy Scan**: Security scans pass without high-severity issues.
-   **Build and Docker Image Creation**: The application is built, and a Docker image is created.
-   **Push to Docker Hub**: The Docker image is pushed successfully.
-   **Deploy to EKS**: The application is deployed and accessible through the load balancer.
  ![3](https://github.com/user-attachments/assets/022eff1b-895a-4cd9-acde-0145706b6eed)



## 🌐Accessing the Deployed Application

Navigate to the **AWS Management Console > EC2 > Load Balancers** to find the DNS name of the load balancer. Use this URL to access your deployed application.
