pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/arfaouinadhar/ci-cd-demo.git'
            }
        }

        stage('Install Sonar Scanner') {
            steps {
                sh '''
                    # Télécharger et installer sonar-scanner manuellement
                    wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
                    unzip -q sonar-scanner-cli-5.0.1.3006-linux.zip
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        ./sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner \
                        -Dsonar.projectKey=ci-cd-demo \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                // CORRECTION : Augmenter le timeout à 5 minutes
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Build Docker") {
            steps {
                sh '''
                    docker build -t ci-cd-demo:latest .
                '''
            }
        }

        stage("Run Docker") {
            steps {
                sh '''
                    docker stop ci-cd-demo || true
                    docker rm ci-cd-demo || true
                    docker run -d --name ci-cd-demo ci-cd-demo:latest
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline CI/CD terminé!'
        }
        success {
            echo '✅ SUCCÈS: Application déployée avec analyse qualité!'
        }
    }
}
