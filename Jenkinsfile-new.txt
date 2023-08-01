pipeline {
    agent any

    environment {
        // Define environment variables as needed
        DOCKER_REGISTRY = '502746322071.dkr.ecr.ap-south-1.amazonaws.com/demo-project'
        DOCKER_IMAGE_NAME = 'myapp'
        AWS_REGION = 'ap-south-1'
        AWS_ECR_CREDENTIALS = 'AWS Credentials for ECR' // Jenkins credential ID for AWS credentials
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from the Git repository
                git 'https://github.com/Nagendra206/srikanthtask.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                script {
                    docker.withRegistry("${DOCKER_REGISTRY}", "${AWS_ECR_CREDENTIALS}") {
                        def customImage = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'AWS Credentials for ECR', usernameVariable: 'AKIAXKDQECSL3V2Z6I57', passwordVariable: 'Ti7RTO9BnUtkjMbxj9eHEFYTps7LNzwhHmBEFa00')
            ]) {
                // Log in to ECR using Docker login command
                sh "docker login --username AWS --password-stdin ${DOCKER_REGISTRY}" << EOF
                    ${AWS_ACCESS_KEY_ID}
                    ${AWS_SECRET_ACCESS_KEY}
                EOF

                // Tag the built Docker image with ECR repository URL
                sh "docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_ID} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.BUILD_ID}"

                // Push the Docker image to ECR
                sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.BUILD_ID}"
              }
           }
        }


    post {
        always {
            // Clean workspace and perform cleanup tasks
            cleanWs()
        }
    }
}
