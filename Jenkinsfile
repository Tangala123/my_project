pipeline {
    agent any
    environment {
        AWS_REGION = 'eu-west-1'
        ECR_REPOSITORY = 'my-ecr-repo'
        IMAGE_NAME = 'my-app-image'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = 'Account-id'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'Git-Hub', url: 'https://github.com/Tangala123/my_project.git'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "🔍 Scanning image with Trivy..."
                    trivy image --exit-code 1 --severity HIGH,CRITICAL --format table --output trivy-report.txt ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Authenticate Docker to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS \
                        --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }
        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS'
                ]]) {
                    sh """
                        aws eks --region ${AWS_REGION} update-kubeconfig --name my-eks-cluster
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                    """
                }
            }
        }
    }
}
