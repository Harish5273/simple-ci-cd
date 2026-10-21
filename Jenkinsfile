pipeline {
    agent any

    environment {
        DOCKER_HUB_USERNAME = 'harish5273'
        IMAGE_NAME          = 'simple-webpage'
        IMAGE_TAG           = "${BUILD_NUMBER}"
        DOCKER_CRED_ID      = 'docker-hub-credentials'
        K8S_NAMESPACE       = 'simple-webapp'
        DEPLOYMENT_NAME     = 'simple-webapp'
        FULL_IMAGE_NAME     = "${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${FULL_IMAGE_NAME} ."
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${FULL_IMAGE_NAME}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "kubectl create namespace ${K8S_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
                
                // Automatically updates the image tag in deployment.yaml to match the current BUILD_NUMBER
                sh "sed -i 's|image:.*|image: ${FULL_IMAGE_NAME}|' k8s/deployment.yaml"
                
                sh "kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}"
                sh "kubectl apply -f k8s/service.yaml -n ${K8S_NAMESPACE}"
                
                sh "kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=120s"
            }
        }
    }

    post {
        success {
            echo 'Pipeline successful!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}