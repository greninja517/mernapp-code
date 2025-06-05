pipeline{
    agent any

    environment{
        GITHUB_REPO_URL = "https://github.com/greninja517/mernapp-code.git"
        GIT_DEFAULT_BRANCH = "main"

        // Fixed: Removed leading tab character
        GCP_PROJECT_ID = "devops-461410"
        GCP_REGION = "asia-south1"

        ARTIFACT_REGISTRY_FRONTEND_REPO = "mernapp-frontend"
        ARTIFACT_REGISTRY_BACKEND_REPO = "mernapp-backend"

        FRONTEND_IMAGE_NAME = "frontend"
        BACKEND_IMAGE_NAME = "backend"
        IMAGE_TAG = "${BUILD_NUMBER}"

        FRONTEND_DOCKERFILE_DIR = "./frontend"
        BACKEND_DOCKERFILE_DIR = "./backend"

        REGISTRY_URL = "${GCP_REGION}-docker.pkg.dev"
        // Fixed: Using correct repository variables
        FRONTEND_IMAGE_URL = "${REGISTRY_URL}/${GCP_PROJECT_ID}/${ARTIFACT_REGISTRY_FRONTEND_REPO}/${FRONTEND_IMAGE_NAME}"
        BACKEND_IMAGE_URL = "${REGISTRY_URL}/${GCP_PROJECT_ID}/${ARTIFACT_REGISTRY_BACKEND_REPO}/${BACKEND_IMAGE_NAME}"

        FRONTEND_BUILD_ARG_KEY = "REACT_APP_BACKEND_URL"
        FRONTEND_BUILD_ARG_VALUE = "http://file.cloud-ninja.xyz/api/tasks"
    }

    stages{
        stage("GIT CHECKOUT"){
            steps{
                // Cleaning the workspace
                cleanWs()

                echo "-----------Checking out the Source Code-------------"
                git branch: "${env.GIT_DEFAULT_BRANCH}", url: "${env.GITHUB_REPO_URL}" 
            }
        }

        stage("GCP AUTHENTICATION"){
            steps{
                echo "------------Authenticating with GCP------------"
                withCredentials([file(credentialsId: 'gcp_sa_key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh "gcloud auth configure-docker ${env.REGISTRY_URL}"
                }
                echo "-----Authentication Successful-------"
            }
        }

        stage("VALIDATING DOCKERFILES EXISTENCE"){
            steps{
                echo "--------Checking the Dockerfiles existence--------"
                script{
                    def frontendDockerfile = "${env.FRONTEND_DOCKERFILE_DIR}/Dockerfile"
                    def backendDockerfile = "${env.BACKEND_DOCKERFILE_DIR}/Dockerfile"

                    if (!fileExists(frontendDockerfile)) {
                        error("Frontend Dockerfile not found at ${frontendDockerfile}")
                    }
                    if (!fileExists(backendDockerfile)) {
                        error("Backend Dockerfile not found at ${backendDockerfile}")
                    }
                }
                echo "--------Dockerfiles are present--------"
            }
        }

        stage("BUILD FRONTEND AND PUSH TO ARTIFACT REGISTRY"){
            steps{
                echo "-----Building the frontend Image------"
                script{
                    def frontendImage = "${env.FRONTEND_IMAGE_URL}:${env.IMAGE_TAG}"
                    sh "docker build --build-arg ${env.FRONTEND_BUILD_ARG_KEY}=${env.FRONTEND_BUILD_ARG_VALUE} -t ${frontendImage} ${env.FRONTEND_DOCKERFILE_DIR}"

                    echo "------Pushing Frontend Image to Artifact Registry--------"
                    sh "docker push ${frontendImage}"

                    echo "------Frontend Image Pushed Successfully-------"
                }
            }
        }

        stage("BUILD BACKEND AND PUSH TO ARTIFACT REGISTRY"){
            steps{
                echo "-----Building the Backend Image------"
                script{
                    def backendImage = "${env.BACKEND_IMAGE_URL}:${env.IMAGE_TAG}"
                    sh "docker build -t ${backendImage} ${env.BACKEND_DOCKERFILE_DIR}"

                    echo "------Pushing Backend Image to Artifact Registry--------"
                    sh "docker push ${backendImage}"

                    echo "------Backend Image Pushed Successfully-------"
                }
            }
        }
    }

    post{
        always {
            echo "Cleaning up the workspace"
            cleanWs()
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
    }
}