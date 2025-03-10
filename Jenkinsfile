pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                # Check Docker availability
                echo "Docker version:"
                docker --version
                
                # Check directory structure
                echo "Directory structure:"
                ls -la
                '''
            }
        }

        stage('Build & Deploy Backend') {
            steps {
                sh '''
                # Navigate to backend directory (use the direct path, not nested)
                cd backend
                
                # Verify working directory
                echo "Current directory: $(pwd)"
                echo "Backend directory contents:"
                ls -la
                
                # Create a Dockerfile
                echo "Creating Dockerfile..."
                cat > Dockerfile << 'EOF'
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 8080
CMD ["npm", "start"]
EOF
                
                # Verify Dockerfile was created
                echo "Dockerfile contents:"
                cat Dockerfile
                
                # Build the container
                echo "Building Docker image..."
                docker build -t backend-app . || { echo "Docker build failed with exit code $?"; exit 1; }
                
                # Stop existing container if it exists
                echo "Stopping any existing containers..."
                docker stop backend-app || echo "No container to stop"
                docker rm backend-app || echo "No container to remove"
                
                # Run the new container
                echo "Starting container..."
                docker run -d --name backend-app -p 5000:8080 backend-app || { echo "Docker run failed with exit code $?"; exit 1; }
                
                # Check container status
                echo "Container status:"
                docker ps | grep backend-app || echo "Container not showing in docker ps"
                '''
            }
        }

        stage('Test Endpoint') {
            steps {
                sh '''
                # Wait for application to start
                echo "Waiting for application to start..."
                sleep 10
                
                # Check container logs
                echo "Container logs:"
                docker logs backend-app
                
                # Test endpoint with more detailed output
                echo "Testing endpoint..."
                curl -v http://localhost:5000 || echo "Curl failed with exit code $?"
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}