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

### Accessing SonarQube

1.  Open your browser and navigate to **http://localhost:9000**.
2.  Log in to SonarQube with the default credentials (**Username**: `admin`, **Password**: `admin`). You will be prompted to change the password after the first login.
   ![9](https://github.com/user-attachments/assets/1b237001-1823-43e4-9282-9f8909a4f38b)

### Creating a Token

1.  Go to **Administration > Security > Users**.
2.  Select your user profile and navigate to **Tokens**.
3.  Click on **Generate Token**, provide a name for the token, and save it securely.
   
   ![8](https://github.com/user-attachments/assets/7fcd292f-1864-4a1c-903e-c6e7366a216d)

### Adding the Token to Jenkins

1.  In Jenkins, go to **Manage Jenkins > Credentials > System**.
2.  Add the token as a **Secret Text** credential with a recognizable ID (e.g., `sonar-token`).
   
   ![7](https://github.com/user-attachments/assets/4115d4bd-7974-4a7b-bea2-af043858b3c2)

### Configuring SonarQube in Jenkins

1.  Go to **Manage Jenkins > Configure System**.

2.  Scroll down to **SonarQube Servers** and add a new server configuration:
    
    -   **Name**: Provide a descriptive name (e.g., `Local SonarQube`).
    -   **Server URL**: `http://localhost:9000`
    -   **Server Authentication Token**: Select the token credential you created earlier.
3.  Save the configuration.

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

  ![Screenshot 2024-11-05 233800](https://github.com/user-attachments/assets/d3d3326c-cdcf-447d-aa26-4420f5e1ce95)

## 🌐Accessing the Deployed Application

### Option 1: AWS Management Console

1.  Navigate to **AWS Management Console > EC2 > Load Balancers**.
2.  Locate the load balancer associated with your EKS deployment.
3.  Copy the **DNS name** of the load balancer.
4.  Paste the DNS name into your browser to access your application.

### Option 2: Command Line (Using `kubectl` in Jenkins Container)

1.  Access your Jenkins container:
    ```
    docker exec -it jenkins bash
    ```
2.  Run the following command to get the service details:
    ```    
    `kubectl get svc
    ```

3.  Look for the **EXTERNAL-IP** under the **LoadBalancer** type service.
    
4.  Copy the **EXTERNAL-IP** or URL.

   ![5](https://github.com/user-attachments/assets/9bf14c78-701c-473b-b2fc-a4fca2227bc7)
    
6.  Paste the URL into your browser to access your application.
   
   ![6](https://github.com/user-attachments/assets/92454859-0116-469d-a35f-1e553c1a1572)

   
