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

        stage('Debug') {
            steps {
                sh 'whoami && id'
                sh 'ls -ld /var/lib/jenkins'
                sh 'ls -l /var/lib/jenkins/for-personal.pem'
                sh 'cat /var/lib/jenkins/for-personal.pem | head -n 1'
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
                    def nodes = ['172.31.90.196', '172.31.94.94']
                    for (node in nodes) {
                        sh "ssh -i /var/lib/jenkins/for-personal.pem -o StrictHostKeyChecking=no admin@${node} 'sudo ctr -n k8s.io images import -' < java-app.tar"
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
