pipeline {
    agent any

    tools {
        jdk 'JDK-21'
    }

    stages {
        stage('Verify Java') {
            steps {
                sh 'java -version'
            }
        }

        stage('Hello') {
            steps {
                echo 'Hello Jenkins'
            }
        }
    }
}