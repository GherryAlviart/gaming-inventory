pipeline {
    agent any
    environment {
        DOCKER_HUB_USER = 'ghryalvrt'
        IMAGE_BACKEND = "${DOCKER_HUB_USER}/gaming-backend"
        IMAGE_FRONTEND = "${DOCKER_HUB_USER}/gaming-frontend"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Images') {
            steps {
                sh "docker build -t ${IMAGE_BACKEND}:latest ./backend"
                sh "docker build -t ${IMAGE_FRONTEND}:latest ./frontend"
            }
        }
        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${IMAGE_BACKEND}:latest"
                    sh "docker push ${IMAGE_FRONTEND}:latest"
                }
            }
        }
        stage('Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-aks', variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f gaming-k8s.yaml"
                    sh "kubectl apply -f gaming-ingress.yaml"
                    sh "kubectl rollout restart deployment/backend-gaming"
                    sh "kubectl rollout restart deployment/frontend-gaming"
                }
            }
        }
    }
}