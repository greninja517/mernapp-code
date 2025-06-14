pipeline {
    agent any

    environment {
        GITHUB_REPO_URL = "https://github.com/greninja517/mernapp-code.git"
        GIT_DEFAULT_BRANCH = "main"

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

        FRONTEND_IMAGE_URL = "${REGISTRY_URL}/${GCP_PROJECT_ID}/${ARTIFACT_REGISTRY_FRONTEND_REPO}/${FRONTEND_IMAGE_NAME}"
        BACKEND_IMAGE_URL = "${REGISTRY_URL}/${GCP_PROJECT_ID}/${ARTIFACT_REGISTRY_BACKEND_REPO}/${BACKEND_IMAGE_NAME}"

        // Frontend build argument
        FRONTEND_BUILD_ARG_KEY = "REACT_APP_BACKEND_URL"
        FRONTEND_BUILD_ARG_VALUE = "http://app.cloud-ninja.xyz/api/tasks"

        GITOPS_BRANCH = "main"
        GITOPS_HELM_VALUES_PATH = "mern-app/values.yaml"
        GITOPS_REPO_URL = "https://github.com/greninja517/mernapp-gitops.git"
    }

    stages {
        stage("GIT CHECKOUT") {
            steps {
                cleanWs()
                echo "-----------Checking out the Source Code-------------"
                git branch: "${env.GIT_DEFAULT_BRANCH}", url: "${env.GITHUB_REPO_URL}"
            }
        }

        stage("VALIDATE DOCKERFILES") {
            steps {
                echo "--------Checking Dockerfiles Existence--------"
                script {
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

        stage("BUILD & PUSH FRONTEND") {
            steps {
                echo "-----Building & Pushing Frontend Image------"
                withCredentials([file(credentialsId: 'gcp_sa_key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud auth configure-docker $REGISTRY_URL --quiet

                        FRONTEND_IMAGE="$FRONTEND_IMAGE_URL:$IMAGE_TAG"

                        docker build --build-arg $FRONTEND_BUILD_ARG_KEY=$FRONTEND_BUILD_ARG_VALUE \
                                     -t $FRONTEND_IMAGE $FRONTEND_DOCKERFILE_DIR

                        docker push $FRONTEND_IMAGE
                    '''
                }
            }
        }

        stage("BUILD & PUSH BACKEND") {
            steps {
                echo "-----Building & Pushing Backend Image------"
                withCredentials([file(credentialsId: 'gcp_sa_key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud auth configure-docker $REGISTRY_URL --quiet

                        BACKEND_IMAGE="$BACKEND_IMAGE_URL:$IMAGE_TAG"

                        docker build -t $BACKEND_IMAGE $BACKEND_DOCKERFILE_DIR

                        docker push $BACKEND_IMAGE
                    '''
                }
            }
        }

        stage("UPDATE GitOps values.yaml") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'gitops_repo_creds', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    script {
                        def gitopsRepoAuthURL = "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/greninja517/mernapp-gitops.git"
                        sh """
                            git clone -b ${env.GITOPS_BRANCH} ${gitopsRepoAuthURL} gitops-repo
                            cd gitops-repo

                            if [ ! -x "./yq" ]; then
                                curl -sL https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -o yq
                                chmod +x yq
                            fi

                            ./yq e '.frontend.image.tag = "${env.IMAGE_TAG}"' -i ${env.GITOPS_HELM_VALUES_PATH}
                            ./yq e '.backend.image.tag = "${env.IMAGE_TAG}"' -i ${env.GITOPS_HELM_VALUES_PATH}

                            git config user.name "jenkins-server"
                            git config user.email "jenkins-server@gmail.com"
                            git add ${env.GITOPS_HELM_VALUES_PATH}
                            git commit -m "Update image tags to ${env.IMAGE_TAG}"
                            git push origin ${env.GITOPS_BRANCH}
                        """

                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
