pipeline{
    agent any
    tools {
        jdk 'java-iti'
        maven 'maven-iti'
    }

    environment{
        SCANNER_HOME= tool "sonar-scanner"
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
                sh "trivy fs --security-checks vuln,config /var/jenkins_home/workspace/project-1"
            }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }

        stage("Build Docker Image"){
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'docker build -t ahmedtobar/cicd:$BUILD_ID .'
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push ahmedtobar/cicd:$BUILD_ID'
                }
            }
        }

    }
}
