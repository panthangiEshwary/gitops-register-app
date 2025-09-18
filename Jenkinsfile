pipeline {
    agent any

    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = "dockerhub"     // DockerHub creds in Jenkins
        GITOPS_REPO = "https://github.com/panthangiEshwary/gitops-register-app.git"
        GITHUB_CREDENTIALS = "github"           // GitHub PAT creds in Jenkins
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout App Repo") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/panthangiEshwary/register-app.git'
            }
        }

        stage("Build Java App") {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t panthangi/${APP_NAME}:${IMAGE_TAG} ./webapp"
            }
        }

        stage("Push Docker Image") {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                 usernameVariable: "DOCKER_USER",
                                                 passwordVariable: "DOCKER_PASS")]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push panthangi/${APP_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage("Checkout GitOps Repo") {
            steps {
                dir("gitops") {
                    git branch: 'main',
                        credentialsId: "${GITHUB_CREDENTIALS}",
                        url: "${GITOPS_REPO}"
                }
            }
        }

        stage("Update Deployment Tags") {
            steps {
                dir("gitops") {
                    sh """
                       sed -i 's#image: panthangi/${APP_NAME}:.*#image: panthangi/${APP_NAME}:${IMAGE_TAG}#' deployment.yaml
                       cat deployment.yaml
                    """
                }
            }
        }

        stage("Push Deployment Changes") {
            steps {
                dir("gitops") {
                    sh """
                       git config user.name "Eshwary"
                       git config user.email "Eshwary@gmail.com"
                       git add deployment.yaml
                       git commit -m "Updated Deployment to ${IMAGE_TAG}" || echo "No changes to commit"
                    """
                    withCredentials([usernamePassword(credentialsId: "${GITHUB_CREDENTIALS}",
                                                     usernameVariable: "GIT_USER",
                                                     passwordVariable: "GIT_TOKEN")]) {
                        sh """
                           git push https://${GIT_USER}:${GIT_TOKEN}@github.com/panthangiEshwary/gitops-register-app.git main
                        """
                    }
                }
            }
        }
    }
}

