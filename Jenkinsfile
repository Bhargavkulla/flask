pipeline {
    agent any

    environment {
        IMAGE_NAME = 'bhargavakulla/app'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-dev-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Bhargavkulla/microservices-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Configure Kubeconfig') {
            steps {
                withCredentials([string(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG_CONTENT')]) {
                    sh '''
                        mkdir -p $HOME/.kube
                        echo "$KUBECONFIG_CONTENT" > $HOME/.kube/config
                        chmod 600 $HOME/.kube/config
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl rollout status deployment/bhargav-deploy
                '''
            }
        }
    }
}

