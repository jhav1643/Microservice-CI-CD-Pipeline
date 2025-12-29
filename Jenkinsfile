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
                withCredentials([string(credentialsId: 'APP_SECRET', variable: 'APP_SECRET')]) {
                    sh '''
                    echo "Secret loaded securely"
                    '''
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
                failure {
                    echo '❌ Backend tests failed. Fix tests before deploy.'
                }
            }
        }

        stage('Run Containers') {
            steps {
                sh '''
                docker rm -f my-backend my-frontend || true
                docker network create my-network || true

                docker run -d --name my-backend \
                  --network my-network \
                  my-backend-image:latest

                docker run -d --name my-frontend \
                  -p 8080:80 \
                  --network my-network \
                  my-frontend-image:latest
                '''
            }
        }
    }
}
