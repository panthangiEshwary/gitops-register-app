pipeline {
    agent any   // Use any available node

    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage("Cleanup Workspace") { steps { cleanWs() } }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/panthangiEshwary/gitops-register-app.git'
            }
        }
        stage("Update the Deployment Tags") {
            steps {
                sh """
                   sed -i "s|image: ${APP_NAME}:.*|image: ${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml
                """
            }
        }
        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "Eshwary"
                   git config --global user.email "Eshwary@gmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest with ${IMAGE_TAG}" || echo "No changes to commit"
                """
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/panthangiEshwary/gitops-register-app.git main"
                }
            }
        }
        stage("Deploy to Kubernetes") {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-id']) {
                    sh "kubectl apply -f deployment.yaml"
                }
            }
        }
    }
}

