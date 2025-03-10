pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
            }
        }
        stage('Build & Deploy') {
            steps {
                sh '''
                cd backend
                
                # Create a Dockerfile
                cat > Dockerfile << 'EOF'
                FROM node:18
                WORKDIR /app
                COPY . .
                RUN npm ci
                EXPOSE 8080
                CMD ["npm", "start"]
                EOF
                
                # Build and run the container
                docker build -t backend-app .
                
                # Stop existing container if it exists
                docker stop backend-app || true
                docker rm backend-app || true
                
                # Run the new container
                docker run -d --name backend-app -p 5000:8080 backend-app
                
                # Check if it's running
                sleep 5
                curl -s http://localhost:5000 || echo "App failed to start"
                '''
            }
        }
    }
}