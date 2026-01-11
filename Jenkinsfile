pipeline {

    agent any

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

        stage('Login + Tag + Push to DockerHub') {
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
                dir('backend') {
                    sh '''
                        echo "Running Jest tests inside container..."
                        docker run --rm my-backend-image:latest npm test || true
                    '''
                }
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
                    docker network rm my-network || true

                    docker network create my-network

                    docker run -d --name my-backend --network my-network my-backend-image:latest

                    docker run -d --name my-frontend -p 8081:80 --network my-network my-frontend-image:latest

                """
            }
        }
    }
}
