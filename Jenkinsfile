pipeline {
    agent any

    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
        }
    }
}
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
                backend=$(docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' backend-app)
                echo "Backend IP address: $backend"
                frontend=$(docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' frontend-app)
                echo "Frontend IP address: $frontend"
                echo "Testing backend..."
                curl -sSf http://$backend:8080 || exit 1
                curl -sSf http://$frontend:3000 || exit 1
                '''
            }
        }
        stage('build image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
        }
                sh '''
                sleep 10
                cd backend
                echo "Building backend image"
                docker image tag backend-app kolizo12/backend-app:$BUILD_NUMBER
                docker image push kolizo12/backend-app:$BUILD_NUMBER
                cd ../frontend
                echo "Building frontend image"
                docker image tag frontend-app kolizo12/frontend-app:$BUILD_NUMBER
                docker image push kolizo12/frontend-app:$BUILD_NUMBER
                '''
            }
        }

    }
}