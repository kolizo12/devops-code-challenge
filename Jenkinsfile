pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                echo 'Installing dependencies...'
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
                sh "cd backend && npm audit fix && npm ci"
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
                sh "cd backend && PORT=5000 npm start"
            }
        }
    }
}