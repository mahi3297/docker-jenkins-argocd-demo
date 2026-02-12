pipeline {
    agent any

    environment {
        // Docker Hub creds stored in Jenkins
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        // Your Docker Hub image name
        IMAGE_NAME = 'mahender397/jenkins-argocd-demo'
        // Kubernetes manifest Git repo (GitOps repo)
        MANIFEST_REPO = 'https://github.com/mahi3297/k8s-manifests.git'
        // Branch of manifest repo (main by default)
        MANIFEST_BRANCH = 'main'
    }

    stages {

        stage('Checkout App') {
            steps {
                echo "Cloning application code..."
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/mahi3297/docker-jenkins-argocd-demo.git']]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing Python dependencies..."
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running unit tests..."
                sh 'pytest'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
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
                echo "Cloning manifest repo and updating image tag..."
                sh """
                rm -rf manifest-repo
                git clone -b ${MANIFEST_BRANCH} ${MANIFEST_REPO} manifest-repo
                cd manifest-repo

                # Update image in deployment.yaml
                sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml

                git config user.email "jenkins@ci.com"
                git config user.name "Jenkins CI"
                git add deployment.yaml
                git commit -m "chore: update image tag to ${IMAGE_TAG}"
                git push
                """
            }
        }

    }

    post {
        always {
            echo "Cleaning up workspace..."
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

