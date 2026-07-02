pipeline {
    agent any

    tools {
        jdk 'JDK-21'
        maven 'Maven-3'
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

        stage('Hello') {
            steps {
                echo 'Hello Jenkins'
            }
        }
    }
}