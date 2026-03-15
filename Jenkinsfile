pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "java-app:latest"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh 'kubectl apply -f src/main/resources/k8s/deployment.yml'
            }
        }
    }

    post {
        success {
            echo 'Application successfully built and deployed!'
        }
    }
}
