pipeline {
    agent any

    tools {
        sonarQubeScanner'sonarScanner'   
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'sonar-scanner -Dsonar.projectKey=demo-kali -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=$SONAR_TOKEN'
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

        stage('Build & Run Docker') {
            steps {
                sh 'docker build -t ci-cd-demo:latest .'
                sh 'docker stop ci-cd-demo || true && docker rm ci-cd-demo || true'
                sh 'docker run -d --name ci-cd-demo ci-cd-demo:latest'
            }
        }
    }
}
