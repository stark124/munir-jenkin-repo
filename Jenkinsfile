pipeline {

    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'munir-jenkin-repo'
        EKS_CLUSTER = 'alchemy-eks-jenkins'
        AWS_ACCOUNT_ID = '992382458064'

        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password \
                  --region ${AWS_REGION} | \
                docker login \
                  --username AWS \
                  --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker tag \
                  ${ECR_REPO}:${IMAGE_TAG} \
                  ${IMAGE_URI}

                docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                  --region ${AWS_REGION} \
                  --name ${EKS_CLUSTER}

                sed "s|IMAGE_URI|${IMAGE_URI}|g" deployment.yaml | \
                kubectl apply -f -

                kubectl apply -f service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl rollout status deployment/jenkins-nginx

                kubectl get deployment jenkins-nginx
                kubectl get pods
                kubectl get service jenkins-nginx-service
                '''
            }
        }
    }
}
