pipeline {
    agent any
    environment {
        IMAGE_NAME = "my-flask-app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        AWS_REGION = "us-east-1"
        ECR_REPO = "851725352687.dkr.ecr.us-east-1.amazonaws.com/my-flask-app"
        KUBE_DEPLOYMENT_FILE = "deployment.yml"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'whoami'
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}'
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}'
                    sh 'docker push ${ECR_REPO}:${IMAGE_TAG}'
                }
            }
        }         
        stage('Deploy to EKS') {
            steps {
                script {

                    sh '''
                        echo "Updating Kubernetes deployment file..."
                        sed -i "s|image: .*|image: ${ECR_REPO}:${IMAGE_TAG}|" ${KUBE_DEPLOYMENT_FILE}

                        echo "Applying Kubernetes deployment..."
                        kubectl apply -f ${KUBE_DEPLOYMENT_FILE}
                    '''
                }
            }
        }
    }
}
