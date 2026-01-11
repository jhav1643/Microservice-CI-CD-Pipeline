pipeline {
    agent {
        label 'docker-linux'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Load Secrets') {
            steps {
                withCredentials([string(credentialsId: 'app-secret', variable: 'app-secret')]) {
                    sh 'echo Secret loaded securely'
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

        stage('Login to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh """
                        echo \$PASS | docker login -u \$USER --password-stdin
                    """
                }
            }
        }

        stage('Tag Images') {
            steps {
                sh """
                docker tag my-frontend-image:latest $USER/my-frontend:${BUILD_NUMBER}
                docker tag my-backend-image:latest $USER/my-backend:${BUILD_NUMBER}
                """
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push $USER/my-frontend:${BUILD_NUMBER}
                docker push $USER/my-backend:${BUILD_NUMBER}
                """
            }
        }

        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh 'pytest tests/ --junitxml=report.xml'
                }
            }
            post {
                always {
                    junit 'report.xml'
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
