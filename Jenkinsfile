pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mahender397/jenkins-argocd-demo"
        DOCKER_CREDENTIALS = "dockerhub-creds"   // change if different
        GITHUB_CREDENTIALS = "GitHub Token"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                        docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Run Tests Inside Container') {
            steps {
                sh """
                    docker run --rm ${DOCKER_IMAGE}:${env.BUILD_NUMBER} pytest
                """
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS}",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Update GitOps Manifest Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${GITHUB_CREDENTIALS}",
                        usernameVariable: "GIT_USER",
                        passwordVariable: "GIT_PASS"
                    )]) {

                        sh """
                            rm -rf manifest-repo
                            git clone -b main https://${GIT_USER}:${GIT_PASS}@github.com/mahi3297/k8s-manifests.git manifest-repo
                            cd manifest-repo

                            sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' deployment.yaml

                            git config user.email "jenkins@ci.com"
                            git config user.name "Jenkins CI"

                            git add deployment.yaml
                            git commit -m "update image to ${env.BUILD_NUMBER}"
                            git push origin main
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        always {
            cleanWs()
        }
    }
}
