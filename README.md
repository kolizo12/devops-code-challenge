# DevOps Code Challenge

This repository contains a containerized application with CI/CD pipelines for deployment to both local Docker environments and Amazon EKS (Elastic Kubernetes Service).

## Project Structure

```
devops-code-challenge/
├── backend/             # Node.js backend application
├── frontend/            # React frontend application
├── Jenkinsfile          # CI/CD pipeline for local Docker deployment
├── Jenkinsfile2         # CI/CD pipeline for EKS deployment
└── main.tf           # Infrastructure as Code for EKS cluster
```

## Application Components

### Backend

- Node.js application
- Exposes RESTful API on port 8080
- Containerized with Docker

### Frontend

- React application
- Connects to the backend API
- Containerized with Docker
- Exposed on port 3000

## CI/CD Pipelines

This project uses Jenkins pipelines for continuous integration and deployment.

### Local Docker Deployment (Jenkinsfile)

The first pipeline handles:

1. Building Docker images for both frontend and backend
2. Local deployment for testing
3. Running basic health checks
4. Tagging and pushing images to Docker Hub
5. Triggering the EKS deployment pipeline

```groovy
// Key stages
stage('Build Backend')       // Builds backend Docker image
stage('Deploy Backend')      // Runs backend container locally
stage('Build Frontend')      // Builds frontend Docker image
stage('Deploy Frontend')     // Runs frontend container locally
stage('Test')                // Runs health checks on both services
stage('Build and Push Images') // Pushes images to Docker Hub
stage('Trigger EKS Deployment') // Triggers the EKS pipeline
```

### EKS Deployment (Jenkinsfile2)

The second pipeline handles:

1. Validating deployment parameters
2. Configuring AWS CLI and kubectl
3. Deploying Kubernetes resources to EKS
4. Verifying the deployment

```groovy
// Key stages
stage('Validate Parameters')    // Checks required parameters
stage('Configure kubectl')      // Sets up AWS and kubectl
stage('Deploy to EKS')          // Applies Kubernetes manifests
stage('Verify Deployment')      // Validates the deployment
```

## Kubernetes Resources

The EKS deployment creates the following resources:

- **Deployments**: For both frontend and backend applications
- **Services**: ClusterIP for backend, LoadBalancer for frontend
- **Ingress**: AWS ALB-based ingress for routing traffic

## Infrastructure as Code

The EKS cluster is provisioned using Terraform with the following components:

- VPC with public and private subnets
- EKS cluster with managed node groups
- IAM roles and policies for cluster operation
- Add-ons like CoreDNS and pod identity agent

## Setup and Usage

### Prerequisites

- Jenkins with Docker support
- AWS CLI and kubectl installed
- AWS account with EKS permissions
- Docker Hub account

### Environment Variables

Set the following environment variables in Jenkins:

- `AWS_ACCESS_KEY_ID`: AWS access key
- `AWS_SECRET_ACCESS_KEY`: AWS secret key
- `AWS_REGION`: AWS region (e.g., "us-west-1")
- `EKS_CLUSTER_NAME`: Name of your EKS cluster (e.g., "demo")
- `KUBERNETES_NAMESPACE`: Namespace for deployments (e.g., "my-app")

### Jenkins Credentials

Create the following Jenkins credentials:

- `docker-hub-credentials`: Username/password for Docker Hub

### Running the Pipelines

1. Create a Jenkins pipeline job pointing to `Jenkinsfile`
2. Create a second Jenkins pipeline job pointing to `Jenkinsfile2`
3. Run the first pipeline, which will automatically trigger the second upon completion

### Accessing the Application

After successful deployment, the application will be accessible via:

- The LoadBalancer URL (printed in the pipeline logs)
- The Ingress ALB URL (if configured)

## Terraform Deployment

To deploy the EKS cluster:

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

Configure kubectl to connect to your cluster:

```bash
aws eks update-kubeconfig --name demo --alias demo --region us-west-1
```

## Deployment Verification

After deployment, you can verify the application is running correctly:

### Check Pod Logs

```bash
# View frontend logs
kubectl logs frontend-deployment-8765c697f-sqgdn -f

# Sample output:
> frontend@0.1.0 start
> react-scripts start
(node:25) [DEP_WEBPACK_DEV_SERVER_ON_AFTER_SETUP_MIDDLEWARE] DeprecationWarning: 'onAfterSetupMiddleware' option is deprecated. Please use the 'setupMiddlewares' option.
(Use `node --trace-deprecation ...` to show where the warning was created)
(node:25) [DEP_WEBPACK_DEV_SERVER_ON_BEFORE_SETUP_MIDDLEWARE] DeprecationWarning: 'onBeforeSetupMiddleware' option is deprecated. Please use the 'setupMiddlewares' option.
Starting the development server...
Compiled successfully!
You can now view frontend in the browser.
  Local:            http://localhost:3000
  On Your Network:  http://10.0.26.150:3000
Note that the development build is not optimized.
To create a production build, use npm run build.
webpack compiled successfully
```

### Access the Application

You can access the application via the AWS Load Balancer URL:

```bash
# Get the LoadBalancer URL
kubectl get service frontend-service -n my-app -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Access the application
curl abb9aea1c9c55471dbf9e6a2b7378235-1507234697.us-west-1.elb.amazonaws.com
```

Sample response showing the application is serving correctly:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <title>Lightfeather Test App</title>
  <script defer src="/static/js/bundle.js"></script></head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```
For a werid reason it loadini=g on the browser look problematic

I get a load failed

