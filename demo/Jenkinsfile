pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/riyajoshi2004/Springboot_Application.git'
            }
        }
        stage('Hello') {
            steps {
                echo 'Hello Jenkins'
            }
        }
    }
}