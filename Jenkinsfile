pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
                sh "cd backend && npm ci && npm start"
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
            }
        }
    }
}