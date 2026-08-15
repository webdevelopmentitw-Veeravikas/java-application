pipeline {
    agent any

    environment {
        APP_NAME = 'java-app'
        IMAGE_NAME = 'your-dockerhub-username/java-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        KUBE_CONFIG_ID = 'kubeconfig'
    }

    tools {
        maven 'Maven'
        jdk 'JDK21'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${FULL_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: KUBE_CONFIG_ID, variable: 'KUBE_CONFIG_FILE')]) {
                    sh '''
                        mkdir -p ~/.kube
                        cp "$KUBE_CONFIG_FILE" ~/.kube/config
                        sed -i "s|your-dockerhub-username/java-app:1.0.0|${IMAGE_NAME}:${IMAGE_TAG}|g" kubernetes.yaml
                        kubectl apply -f kubernetes.yaml
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
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Build or deployment failed. Please check logs.'
        }
    }
}
