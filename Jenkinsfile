pipeline {
    agent any

    tools {
        jdk 'JDK-21'
        maven 'Maven-3'
    }

    environment {
        DOCKER_IMAGE = "hello-world-springboot"
        DOCKER_TAG   = "${env.BUILD_NUMBER}"
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

        stage('Hello') {
            steps {
                echo 'Hello Jenkins'
            }
        }
    }
}