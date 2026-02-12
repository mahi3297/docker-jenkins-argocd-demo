# Jenkins CI to Argo CD CD Pipeline

This project demonstrates a complete CI/CD pipeline using:

- GitHub
- Jenkins (CI)
- Docker Hub (Image Registry)
- Kubernetes
- Argo CD (CD)

## Flow

1. Developer pushes code to GitHub
2. Jenkins runs tests, builds Docker image, pushes to Docker Hub
3. Jenkins updates Kubernetes manifest repo with new image tag
4. Argo CD auto-syncs and deploys the application to Kubernetes

## Tech Stack

- Python Flask
- Docker
- Jenkins
- Kubernetes
- Argo CD

## Jenkins Setup

- Add Docker Hub credentials in Jenkins as: `dockerhub-creds`
- Create a Pipeline job pointing to this repo
- Trigger builds via GitHub webhook or manual run

## Output

Access the app via NodePort service after Argo CD deployment.

