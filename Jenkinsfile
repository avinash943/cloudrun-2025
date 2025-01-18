pipeline {
    agent any

    environment {
        // Docker Hub Credentials and Image Name
        DOCKER_HUB_CREDENTIALS_USR = 'afroz2022'   // Your Docker Hub username
        IMAGE_NAME = 'afroz2022/my-go-app'          // Your Docker image name
        DOCKER_HUB_CREDENTIALS_PSWD = credentials('docker-hub-password')  // Your Docker Hub password credentials in Jenkins

        // Google Cloud details
        PROJECT_ID = 'my-first-devops-project-444911' // Your GCP Project ID
        REGION = 'us-central1'                      // Replace with your desired GCP region (default: us-central1)
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    // Checkout code from GitHub repository (main branch)
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }

                    // Push the Docker image to Docker Hub
                    sh "docker push ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Google Cloud Run') {
            steps {
                script {
                    // Authenticate with Google Cloud using the service account key stored in Jenkins
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        // Set GCP project
                        sh "gcloud config set project ${PROJECT_ID}"

                        // Deploy the Docker image to Google Cloud Run
                        sh "gcloud run deploy ${IMAGE_NAME} \
                            --image gcr.io/${PROJECT_ID}/${IMAGE_NAME}:${BUILD_NUMBER} \
                            --platform managed \
                            --region ${REGION} \
                            --allow-unauthenticated"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up the workspace after the job finishes (inside node block)
            node('any') { // Fix the missing 'label' issue by adding 'node('any')'
                cleanWs()
            }
        }
    }
}
