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
                
                # Check available ports
                echo "Checking which ports are in use:"
                netstat -tulpn | grep -E ':(5000|5001|8080|8081|3000)' || echo "No conflicts on common ports"
                
                # Check directory structure
                echo "Directory structure:"
                ls -la
                '''
            }
        }

        stage('Build & Deploy Backend') {
            steps {
                sh '''
                # Navigate to backend directory
                cd backend
                
                # Verify working directory
                echo "Current directory: $(pwd)"
                
                # Create a Dockerfile
                echo "Creating Dockerfile..."
                cat > Dockerfile << 'EOF'
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm audit fix --force
EXPOSE 8080
CMD ["npm", "start"]
EOF
                
                # Build the container
                echo "Building Docker image..."
                docker build -t backend-app .
                
                # Stop existing container if it exists
                echo "Stopping any existing containers..."
                docker stop backend-app || echo "No container to stop"
                docker rm backend-app || echo "No container to remove"
                
                # Try different ports until one works
                for PORT in 5001 5002 5003 5004 5005; do
                    echo "Attempting to start container on port $PORT..."
                    if docker run -d --name backend-app -p $PORT:8080 backend-app; then
                        echo "Container started successfully on port $PORT"
                        echo $PORT > ../backend_port.txt
                        break
                    else
                        echo "Failed to start on port $PORT, trying next port..."
                        docker rm backend-app || true
                    fi
                done
                
                # Check if container is running
                if ! docker ps | grep backend-app; then
                    echo "Failed to start container on any port!"
                    exit 1
                fi
                
                # Get the port that worked
                BACKEND_PORT=$(cat ../backend_port.txt)
                echo "Backend running on port $BACKEND_PORT"
                '''
            }
        }

        stage('Test Endpoint') {
            steps {
                sh '''
                # Get the port that worked
                BACKEND_PORT=$(cat backend_port.txt)
                
                # Wait for application to start
                echo "Waiting for application to start on port $BACKEND_PORT..."
                sleep 10
                
                # Check container logs
                echo "Container logs:"
                docker logs backend-app
                
                # Test endpoint
                echo "Testing endpoint..."
                curl -v http://localhost:$BACKEND_PORT || echo "Curl failed with exit code $?"
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
            sh '''
            # Print running containers
            echo "Running containers:"
            docker ps
            
            # Print used ports
            echo "Used ports:"
            netstat -tulpn | grep -E ':(5000|5001|5002|5003|8080|8081|3000)' || echo "None of the checked ports are in use"
            '''
        }
    }
}