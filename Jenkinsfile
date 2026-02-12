pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mahender397/jenkins-argocd-demo"
        DOCKER_CREDENTIALS = "dockerhub-creds"
        GITHUB_CREDENTIALS = "GitHub Token"
        MANIFEST_REPO = "https://github.com/mahi3297/k8s-manifests.git"
        MANIFEST_BRANCH = "main"
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
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Run Tests Inside Container') {
            steps {
                sh "docker run --rm ${DOCKER_IMAGE}:${env.BUILD_NUMBER} pytest"
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Update GitOps Manifest Repo') {
            steps {
                withCredentials([string(credentialsId: env.GITHUB_CREDENTIALS, variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        rm -rf manifest-repo
                        git clone -b ${MANIFEST_BRANCH} https://${GITHUB_TOKEN}@github.com/mahi3297/k8s-manifests.git manifest-repo
                        cd manifest-repo
                        sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|g" deployment.yaml
                        git config user.email "jenkins@ci.com"
                        git config user.name "Jenkins CI"
                        git add deployment.yaml
                        git commit -m "chore: update image to ${BUILD_NUMBER}"
                        git push origin ${MANIFEST_BRANCH}
                    '''
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
