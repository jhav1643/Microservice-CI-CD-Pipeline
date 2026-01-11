pipeline {

    agent { label 'docker-linux' }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Load Secrets') {
            steps {
                withCredentials([string(credentialsId: 'app-secret', variable: 'APP_SECRET')]) {
                    sh 'echo "Secret loaded securely"'
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t my-frontend-image:latest .'
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t my-backend-image:latest .'
                }
            }
        }

        stage('Login + Tag + Push DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag my-frontend-image:latest $DOCKER_USER/my-frontend:${BUILD_NUMBER}
                        docker tag my-backend-image:latest $DOCKER_USER/my-backend:${BUILD_NUMBER}

                        docker push $DOCKER_USER/my-frontend:${BUILD_NUMBER}
                        docker push $DOCKER_USER/my-backend:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Test Backend') {
            steps {
                sh """
                    docker run --rm \
                        -v $WORKSPACE/backend:/app \
                        my-backend-image:latest \
                        pytest tests/ --junitxml=/app/report.xml || true
                """
            }
            post {
                always {
                    junit 'backend/report.xml'
                }
            }
        }


        stage('Run Containers') {
            steps {
                sh """
                    docker rm -f my-backend my-frontend || true
                    docker network create my-network || true

                    docker run -d --name my-backend --network my-network my-backend-image:latest
                    docker run -d --name my-frontend -p 8080:80 --network my-network my-frontend-image:latest
                """
            }
        }
    }
}
