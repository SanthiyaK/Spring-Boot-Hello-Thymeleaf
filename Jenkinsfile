pipeline {
    agent any
    
    environment {
        DOCKER_BUILDKIT = "1"  // Enable BuildKit
    }
    
    tools {
        maven 'Maven3'     
        jdk 'JDK17'  
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/SanthiyaK/Spring-Boot-Hello-Thymeleaf.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                        mvn deploy -DskipTests \
                        -DaltDeploymentRepository=nexus-snapshots::default::http://16.171.22.123:8081/repository/maven-snapshots/ \
                        -Dnexus.username=$NEXUS_USER \
                        -Dnexus.password=$NEXUS_PASS
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'DOCKER_BUILDKIT=0 docker build -t santhiyakrishdevops/springboot-app:latest .'
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag santhiyakrishdevops/springboot-app:latest $DOCKER_USER/springboot-app:latest
                        docker push $DOCKER_USER/springboot-app:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }
}
