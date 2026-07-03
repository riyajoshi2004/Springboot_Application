pipeline {
    agent any

    tools {
        jdk 'JDK-21'
        maven 'Maven-3'
    }

    environment {
        DOCKER_IMAGE     = "hello-world-springboot"
        DOCKER_TAG       = "${env.BUILD_NUMBER}"
        KUBECONFIG       = "/var/jenkins_home/.kube/config"
        DOCKER_HOST      = "tcp://192.168.49.2:2376"
        DOCKER_TLS_VERIFY = "1"
        DOCKER_CERT_PATH  = "/tmp/.minikube/certs"
    }

    stages {
        stage('Verify Java') {
            steps {
                sh 'java -version'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -version'
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl set image deployment/hello-world-deployment hello-world=${DOCKER_IMAGE}:${DOCKER_TAG} -n staging"
                sh "kubectl rollout status deployment/hello-world-deployment -n staging"
            }
        }

        stage('Hello') {
            steps {
                echo 'Hello Jenkins'
            }
        }
    }
}