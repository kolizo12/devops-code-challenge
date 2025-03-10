pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
            }
        }
        
        stage('Build Backend') {
            steps {
                sh '''
                # Navigate to backend directory
                cd devops-code-challenge/backend
                
                # Stop and remove existing container if any
                docker stop backend-app || true
                docker rm backend-app || true
                
                # Build backend image
                docker build -t backend-app:latest .
                
                # Run backend container on port 5000 instead of 8080
                docker run -d --name backend-app -p 5000:8080 -e CORS_ORIGIN=http://localhost:3000 backend-app:latest
                '''
            }
        }
        
        stage('Build Frontend') {
            steps {
                sh '''
                # Navigate to frontend directory
                cd devops-code-challenge/frontend
                
                # Stop and remove existing container if any
                docker stop frontend-app || true
                docker rm frontend-app || true
                
                # Build frontend image
                docker build -t frontend-app:latest .
                
                # Run frontend container - update API URL to point to port 5000
                docker run -d --name frontend-app -p 3000:3000 -e REACT_APP_API_URL=http://localhost:5000 frontend-app:latest
                '''
            }
        }
        
        stage('Verify') {
            steps {
                sh '''
                # List running containers
                docker ps
                
                # Check if endpoints are responding
                echo "Checking backend status..."
                curl -s http://localhost:5000 || echo "Backend not responding"
                
                echo "Checking frontend status..."
                curl -s http://localhost:3000 || echo "Frontend not responding"
                '''
            }
        }
    }
}