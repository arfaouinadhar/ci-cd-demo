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

        stage('Clean Workspace') {
            steps {
                sh '''
                    # Nettoyer les installations précédentes de Sonar Scanner
                    rm -rf sonar-scanner* .scannerwork
                '''
            }
        }

        stage('Install Sonar Scanner') {
            steps {
                sh '''
                    # Télécharger Sonar Scanner
                    wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
                    
                    # Extraire sans interaction utilisateur
                    unzip -q -o sonar-scanner-cli-5.0.1.3006-linux.zip
                    
                    # Vérifier que l'installation a réussi
                    ls -la sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner
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

        stage('Cleanup') {
            steps {
                sh '''
                    # Nettoyer les fichiers temporaires
                    rm -rf sonar-scanner-cli-*.zip
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
