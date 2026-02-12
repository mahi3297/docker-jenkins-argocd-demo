pipeline {
    agent {
        docker {
            image 'python:3.9'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        IMAGE_NAME = 'mahender397/jenkins-argocd-demo'
        MANIFEST_REPO = 'https://github.com/mahi3297/k8s-manifests.git'
        MANIFEST_BRANCH = 'main'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS,
                                                     usernameVariable: 'DOCKER_USER',
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Update K8s Manifest Repo') {
            steps {
                sh """
                rm -rf manifest-repo
                git clone -b ${MANIFEST_BRANCH} ${MANIFEST_REPO} manifest-repo
                cd manifest-repo
                sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml
                git config user.email "jenkins@ci.com"
                git config user.name "Jenkins CI"
                git add deployment.yaml
                git commit -m "chore: update image to ${IMAGE_TAG}"
                git push
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
