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
                # Create docker-compose file
                cat > docker-compose.yml << EOF
version: '3'

services:
  backend:
    build: ./devops-code-challenge/backend
    ports:
      - "8080:8080"
    environment:
      - CORS_ORIGIN=http://localhost:3000
  
  frontend:
    build: ./devops-code-challenge/frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8080
    depends_on:
      - backend
EOF

                # Create backend Dockerfile
                mkdir -p devops-code-challenge/backend
                cat > devops-code-challenge/backend/Dockerfile << EOF
FROM node:18

WORKDIR /app
COPY . .
RUN npm install
EXPOSE 8080
CMD ["npm", "start"]
EOF

                # Create frontend Dockerfile
                mkdir -p devops-code-challenge/frontend
                cat > devops-code-challenge/frontend/Dockerfile << EOF
FROM node:18

WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
EOF

                # Start services
                docker-compose down || true
                docker-compose up -d
                '''
            }
        }
    }
}