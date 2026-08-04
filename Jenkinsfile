pipeline {

    agent any

    environment {
        IMAGE_NAME   = "frontend"
        IMAGE_TAG    = "build-${BUILD_NUMBER}"
        AWS_REGION   = "us-east-1"
        ECR_REGISTRY = "001634075226.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Git Debug') {
            steps {
                sh '''
                    echo "===== GIT DEBUG ====="
                    pwd
                    git status
                    echo

                    echo "Current branch:"
                    git branch
                    echo

                    echo "All branches:"
                    git branch -a
                    echo

                    echo "HEAD:"
                    git rev-parse --abbrev-ref HEAD
                    echo

                    echo "Remote:"
                    git remote -v
                    echo
                '''
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
                withAWS(credentials: 'aws_creds', region: "${AWS_REGION}") {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login \
                        --username AWS \
                        --password-stdin ${ECR_REGISTRY}
                    """
                }
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
                withAWS(credentials: 'aws_creds', region: "${AWS_REGION}") {
                    sh """
                        docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('GitOps Update') {
            steps {

                sh """
                    echo "Checking out main branch..."
                    git checkout main
                    git pull origin main
                """

                sh """
                    export IMAGE_TAG=${IMAGE_TAG}

                    echo "Updating GitOps manifest..."

                    yq -i '(.images[] | select(.name == "frontend")).newTag = env(IMAGE_TAG)' \
                    deployments/overlays/dev/kustomization.yaml
                """

                sh """
                    echo "Manifest changes:"
                    git diff deployments/overlays/dev/kustomization.yaml
                """

                sh """
                    git config user.name "Jenkins CI"
                    git config user.email "jenkins@example.local"

                    git add deployments/overlays/dev/kustomization.yaml

                    if git diff --cached --quiet; then
                        echo "No manifest changes detected."
                    else
                        git commit -m "Update frontend image to ${IMAGE_TAG}"
                        git push origin main
                    fi
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

    }

}
