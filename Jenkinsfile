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
                docker --version || echo "Docker command failed"
                
                # Check current directory structure
                echo "Directory structure:"
                ls -la
                '''
            }
        }

        stage('Build & Deploy') {
            steps {
                sh '''
                # Verify directory structure
                echo "Root directory contents:"
                ls -la
                
                # Navigate to backend directory
                cd devops-code-challenge/backend || { echo "Failed to navigate to backend directory"; exit 1; }
                
                # Verify working directory
                echo "Current directory: $(pwd)"
                echo "Backend directory contents:"
                ls -la
                
                # Create a Dockerfile with error checking
                echo "Creating Dockerfile..."
                cat > Dockerfile << 'EOF'
FROM node:18
WORKDIR /app
COPY . .
RUN npm ci
EXPOSE 8080
CMD ["npm", "start"]
EOF
                
                # Verify Dockerfile was created
                if [ ! -f Dockerfile ]; then
                    echo "Failed to create Dockerfile"
                    exit 1
                fi
                
                echo "Dockerfile contents:"
                cat Dockerfile
                
                # Build the container with explicit error reporting
                echo "Building Docker image..."
                docker build -t backend-app . || { echo "Docker build failed with exit code $?"; exit 1; }
                
                # Stop existing container if it exists
                echo "Stopping any existing containers..."
                docker stop backend-app || echo "No container to stop"
                docker rm backend-app || echo "No container to remove"
                
                # Run the new container
                echo "Starting container..."
                docker run -d --name backend-app -p 5000:8080 backend-app || { echo "Docker run failed with exit code $?"; exit 1; }
                
                # Verify the container is running
                echo "Container status:"
                docker ps -a | grep backend-app || echo "Container not found in docker ps output"
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
                docker logs backend-app || echo "Could not get container logs"
                
                # Test endpoint with more detailed output
                echo "Testing endpoint..."
                curl -v http://localhost:5000 || echo "Curl failed with exit code $?"
                '''
            }
        }
    }
}