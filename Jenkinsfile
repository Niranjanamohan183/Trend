pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKERHUB_NAMESPACE   = 'niranjanamohan180'
        IMAGE_NAME            = "${DOCKERHUB_NAMESPACE}/trendify"
        IMAGE_TAG              = "${env.BUILD_NUMBER}"
        AWS_REGION             = 'ap-south-1'
        EKS_CLUSTER_NAME       = 'trendify-eks'
        KUBECONFIG_CRED        = 'eks-kubeconfig'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
    steps {
        git branch: 'main', url: 'https://github.com/Niranjanamohan183/Trend.git'
    }
}

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh 'echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin'
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        sed -i "s|niranjanamohan180/trendify:latest|${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl rollout status deployment/trendify-deployment --timeout=120s
                    '''
                }
            }
        }

        stage('Get LoadBalancer URL') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl get svc trendify-service -o wide
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "Build #${env.BUILD_NUMBER} deployed successfully: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Build #${env.BUILD_NUMBER} failed. Check logs above."
        }
    }
}