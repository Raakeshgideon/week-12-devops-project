pipeline {
    agent any

    environment {
        IMAGE_NAME = "dockerhubusername/week12-devops"
        SONARQUBE = "SonarQube"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/username/week12-devops.git'
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                pip install -r requirements.txt
                pytest
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=week12-devops \
                    -Dsonar.sources=. \
                    -Dsonar.python.version=3
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy (Local)') {
            steps {
                sh '''
                docker rm -f week12 || true
                docker run -d --name week12 -p 8081:8080 $IMAGE_NAME
                '''
            }
        }
    }
}
