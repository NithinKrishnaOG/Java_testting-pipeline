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

        stage('Distribution') {
            steps {
                sh "docker save ${DOCKER_IMAGE} > java-app.tar"
                script {
                    def nodes = ['44.206.252.3', '34.201.45.23']
                    for (node in nodes) {
                        sh "ssh -i /var/lib/jenkins/for-personal.pem -o StrictHostKeyChecking=no admin@${node} 'docker load' < java-app.tar"
                    }
                }
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
