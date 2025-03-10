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
                cd backend && ls -ltr

                
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
                ls -l Dockerfile
                cat Dockerfile
                # Build and run the container
                docker build -t backend-app .
                
                # Stop existing container if it exists
                docker stop backend-app || true
                docker rm backend-app || true
                
                # Run the new container
                docker run -d --name backend-app -p 5000:8080 backend-app
                '''
            }
        }
        stage('Test Endpoint') {
            steps {
                script {
                    // Check if the container is running
                    sh "docker ps -a"

                    // Check logs for errors
                    sh "docker logs backend-app || true"

                    // Test endpoint
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000", returnStdout: true).trim()
                    if (response != '200') {
                        error("Application failed to start! HTTP response: ${response}")
                    }
                }
            }
        }
    }
}
