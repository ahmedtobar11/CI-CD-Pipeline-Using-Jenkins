pipeline{
    agent any
    tools {
        jdk 'java-iti'
        maven 'maven-iti'
    }

    environment{
        SCANNER_HOME= tool "sonar-scanner"
        DOCKER_IMAGE = 'ahmedtobar/webapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
        AWS_SESSION_TOKEN = credentials('aws_session_token')
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/ahmedtobar11/CI-CD-Pipeline-Using-Jenkins.git'
            }
        }

        stage("Sonarqube Analysis"){
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevOps-CICD\
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=DevOps-CICD\
                        -Dsonar.login=$SONAR_TOKEN'''
                    }

                }
            }
        }

        stage("Trivy Scan"){
            steps {
                sh "trivy fs --security-checks vuln,config /var/jenkins_home/workspace/CICD"
            }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }

        stage("Build Docker Image"){
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage("Push to DockerHub"){
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                    sh 'docker push ${DOCKER_IMAGE}:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    //sh "sed -i 's|image:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|' k8s/deployment.yaml"
                    sh 'aws eks --region us-east-1 update-kubeconfig --name EKS-deploy-app'
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }
}