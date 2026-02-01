pipeline {
    agent any

    environment {
        PYTHON = "C:\\Users\\raake\\AppData\\Local\\Programs\\Python\\Python311\\python.exe"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                bat "\"%PYTHON%\" -m pip install --upgrade pip"
                bat "\"%PYTHON%\" -m pip install -r requirements.txt"
                bat "\"%PYTHON%\" -m pytest"
            }
        }

        stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('week 12 - devops') {
                    bat """
                    sonar-scanner ^
                    -Dsonar.projectKey=week-12-devops-project ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=http://localhost:9090
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t raakeshdev/week12-devops:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat 'docker push raakeshdev/week12-devops:latest'
                }
            }
        }

        stage('Deploy Local') {
            steps {
                bat 'docker run -d -p 5000:5000 raakeshdev/week12-devops:latest'
            }
        }
    }
}
