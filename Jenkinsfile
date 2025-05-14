pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "umamahesh571/kubeapp:latest"
        KUBE_CONFIG = credentials('kubeconfig') // Jenkins credential (secret file)
        DOCKER_CREDENTIALS_ID = 'dockerhub' // Jenkins credentials for DockerHub
    }

    stages {
        stage('Checkout') {
            steps {
                git branch= 'main' url= 'https://github.com/umamahesh571/springboot-kubeapp.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG}", variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Something went wrong.'
        }
    }
}
