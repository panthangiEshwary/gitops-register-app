pipeline {
    agent any
    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = "dockerhub"
    }
    stages {
        stage("Cleanup Workspace") { steps { cleanWs() } }
        stage("Checkout App Repo") { steps { git branch: 'main', credentialsId: 'github', url: 'https://github.com/panthangiEshwary/register-app.git' } }
        stage("Build Java App") { steps { sh "mvn clean install -DskipTests" } }
        stage("Build Docker Image") { steps { sh "docker build -t panthangi/${APP_NAME}:${IMAGE_TAG} ." } }
        stage("Push Docker Image") {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push panthangi/${APP_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }
        stage("Checkout GitOps Repo") { steps { git branch: 'main', credentialsId: 'github', url: 'https://github.com/panthangiEshwary/gitops-register-app.git' } }
        stage("Update Deployment Tags") {
            steps {
                sh """
                   sed -i 's#image: panthangi/${APP_NAME}:.*#image: panthangi/${APP_NAME}:${IMAGE_TAG}#' deployment.yaml
                   cat deployment.yaml
                """
            }
        }
        stage("Push Deployment Changes") {
            steps {
                sh """
                   git config --global user.name "Eshwary"
                   git config --global user.email "Eshwary@gmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment to ${IMAGE_TAG}" || echo "No changes to commit"
                   git push https://github.com/panthangiEshwary/gitops-register-app.git main
                """
            }
        }
    }
}

