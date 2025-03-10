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

                # Verify working directory
                echo "Current directory: $(pwd)"

                # Create a Dockerfile
                cat > Dockerfile << 'EOF'
                FROM node:18
                WORKDIR /app
                COPY . .
                RUN npm ci
                EXPOSE 8080
                CMD ["npm", "start"]
                EOF

                echo 'Dockerfile created successfully'

                # Verify Dockerfile exists
                ls -l Dockerfile
                cat Dockerfile

                # Build the container
                docker build -t backend-app . || { echo "Docker build failed"; exit 1; }

                # Stop existing container if it exists
                docker stop backend-app || true
                docker rm backend-app || true

                # Run the new container
                docker run -d --name backend-app -p 5000:8080 backend-app || { echo "Docker run failed"; exit 1; }

                # Verify the container is running
                docker ps -a
                '''
            }
        }

        stage('Test Endpoint') {
            steps {
                script {
                    // Verify running containers
                    sh "docker ps -a"

                    // Check logs for errors
                    sh "docker logs backend-app || true"

                    // Test endpoint
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000", returnStdout: true).trim()
                    if (response != '200') {
                        error("Application failed to start! HTTP response: ${response}")
                    } else {
                        echo "Application is running successfully!"
                    }
                }
            }
        }
    }
}
