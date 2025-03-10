pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kolizo12/devops-code-challenge.git', branch: 'main'
            }
        }

        // Backend Stages
        stage('Build Backend') {
            steps {
                sh '''
                cd backend
                docker build -t backend-app -f- . << 'EOF'
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm audit fix 
EXPOSE 8080
CMD ["npm", "start"]
EOF
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                docker stop backend-app || true
                docker rm backend-app || true
                docker run -d --name backend-app -p 8080:8080 -e CORS_ORIGIN=http://localhost:3000 backend-app
                echo "Backend running on port 5001"
                '''
            }
        }

        // Frontend Stages
        stage('Build Frontend') {
            steps {
                sh '''
                cd frontend
                docker build -t frontend-app -f- . << 'EOF'
FROM node:16
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
ENV REACT_APP_API_URL=http://localhost:3000
CMD ["npm", "start"]
EOF
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                docker stop frontend-app || true
                docker rm frontend-app || true
                docker run -d --name frontend-app -p 3000:3000 frontend-app
                echo "Frontend running on port 3000"
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                sleep 10
                echo "Testing backend..."
                curl -s http://localhost:8080 || echo "Backend test failed"
                echo "Testing frontend..."
                curl -s http://localhost:3000 || echo "Frontend test failed"
                '''
            }
        }
    }
}