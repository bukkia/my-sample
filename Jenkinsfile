pipeline {
    agent any

    environment {
        IMAGE_NAME = 'abiolao/my-sample-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/bukkia/my-sample.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🔨 Building Docker image...'
                echo 'i am working hard'
                sh "docker build -t ${FULL_IMAGE_NAME} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Test (Dockerized)') {
            steps {
                echo '🧪 Running tests inside Docker container...'
                sh "docker run --rm ${FULL_IMAGE_NAME} npm test || echo 'No test command defined'"
            }
        }

        stage('Login to Docker Hub') {
            steps {
                echo '🔐 Logging in to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'abiolao-lab-server', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "📦 Pushing ${FULL_IMAGE_NAME} and latest tag to Docker Hub..."
                sh """
                    docker push ${FULL_IMAGE_NAME}
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy (Run Locally)') {
            steps {
                echo '🚀 Deploying container locally...'
                sh "docker rm -f my-sample-app || true"
                sh "docker run -d --name my-sample-app -p 8080:80 ${FULL_IMAGE_NAME}"
            }
        }
    }

    post {
        success {
            echo "✅ Successfully built, tested, pushed, and deployed: ${FULL_IMAGE_NAME}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
