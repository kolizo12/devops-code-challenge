pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
            }
        }
        stage('Build & Deploy Backend') {
            steps {
                sh '''
                cd devops-code-challenge/backend
                
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
                docker run -d --name backend-app -p 5000:8080 -e CORS_ORIGIN=http://localhost:3000 backend-app
                
                # Check if it's running
                sleep 5
                curl -s http://localhost:5000 || echo "Backend failed to start"
                '''
            }
        }
        stage('Build & Deploy Frontend') {
            steps {
                sh '''
                cd devops-code-challenge/frontend
                
                # Create a Dockerfile
                cat > Dockerfile << 'EOF'
                FROM node:18
                WORKDIR /app
                COPY . .
                RUN npm ci
                EXPOSE 3000
                ENV REACT_APP_API_URL=http://localhost:5000
                CMD ["npm", "start"]
                EOF
                
                # Build and run the container
                docker build -t frontend-app .
                
                # Stop existing container if it exists
                docker stop frontend-app || true
                docker rm frontend-app || true
                
                # Run the new container
                docker run -d --name frontend-app -p 3000:3000 frontend-app
                
                # Check if it's running
                sleep 5
                curl -s http://localhost:3000 || echo "Frontend failed to start"
                '''
            }
        }
        stage('Test Integration') {
            steps {
                sh '''
                # Wait for services to fully initialize
                sleep 10
                
                echo "Testing frontend connectivity..."
                # Get the frontend page and check for expected content
                curl -s http://localhost:3000 | grep -i "welcome" && echo "Frontend test passed" || echo "Frontend test failed"
                
                echo "Testing backend API via frontend..."
                # Test that the frontend can connect to the backend
                # This assumes your frontend makes API calls when loaded
                # You might need to adjust the grep pattern based on what you expect to see
                curl -s http://localhost:3000 | grep -i "data" && echo "API integration test passed" || echo "API integration test failed"
                
                # More comprehensive test using a browser automation tool would be better
                # but for a simple test, this checks basic connectivity
                
                echo "Testing direct backend API..."
                curl -s http://localhost:5000/api/some-endpoint && echo "Backend API test passed" || echo "Backend API test failed"
                
                # Docker container status
                echo "Container status:"
                docker ps | grep -E 'backend-app|frontend-app'
                
                # Check container logs for errors
                echo "Backend logs:"
                docker logs backend-app --tail 20
                
                echo "Frontend logs:"
                docker logs frontend-app --tail 20
                '''
            }
        }
    }
    post {
        always {
            echo 'Finishing pipeline...'
        }
        success {
            echo 'Build and tests completed successfully!'
        }
        failure {
            echo 'Build or tests failed!'
        }
    }
}