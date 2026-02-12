pipeline {
    agent any

    environment {
        GITHUB_CREDENTIALS = 'GitHub Token'
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        IMAGE_NAME = 'mahender397/jenkins-argocd-demo'
        MANIFEST_REPO = 'https://github.com/mahi3297/k8s-manifests.git'
        MANIFEST_BRANCH = 'main'
    }

    options {
        timestamps()
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
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Run Tests Inside Container') {
            steps {
                script {
                    sh """
                    docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} pytest || exit 1
                    """
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: DOCKERHUB_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Update GitOps Manifest Repo') {
            steps {
                script {
                    sh """
                    rm -rf manifest-repo
                    git clone -b ${MANIFEST_BRANCH} ${MANIFEST_REPO} manifest-repo
                    cd manifest-repo

                    sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml

                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"

                    git add deployment.yaml
                    git commit -m "chore: update image to ${IMAGE_TAG}"
                    git push origin ${MANIFEST_BRANCH}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline Completed Successfully 🚀"
        }
        failure {
            echo "Pipeline Failed ❌"
        }
        always {
            cleanWs()
        }
    }
}
