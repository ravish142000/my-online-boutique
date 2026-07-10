pipeline {

    agent any

    environment {
        IMAGE_NAME   = "frontend"
        IMAGE_TAG    = "build-${BUILD_NUMBER}"
        AWS_REGION   = "us-east-1"
        ECR_REGISTRY = "811822680488.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                dir('src/frontend') {
                    sh 'go build'
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                dir('src/frontend') {
                    sh 'go test ./...'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('src/frontend') {
                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    docker tag ${IMAGE_NAME}:latest ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push Docker Image to Amazon ECR') {
            steps {
                sh """
                    docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    docker push ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

    }

    post {

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            cleanWs()
        }

    }

}
