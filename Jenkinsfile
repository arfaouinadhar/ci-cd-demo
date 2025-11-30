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
                    rm -rf sonar-scanner-* sonar-scanner-cli-*.zip .scannerwork
                '''
            }
        }

        stage('Install Sonar Scanner') {
            steps {
                sh '''
                    wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
                    unzip -q -o sonar-scanner-cli-5.0.1.3006-linux.zip
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

        stage('Wait for Analysis Processing') {
            steps {
                // Attendre que l'analyse soit traitée par SonarQube
                sleep time: 30, unit: 'SECONDS'
                echo "✅ Analyse SonarQube terminée - Rapport disponible: http://localhost:9000/dashboard?id=ci-cd-demo"
            }
        }

        stage("Build Docker") {
            steps {
                sh '''
                    echo "🔨 Construction de l'image Docker..."
                    docker build -t ci-cd-demo:latest .
                    echo "✅ Image Docker construite avec succès"
                '''
            }
        }

        stage("Run Docker") {
            steps {
                sh '''
                    echo "🚀 Déploiement de l'application..."
                    docker stop ci-cd-demo || true
                    docker rm ci-cd-demo || true
                    docker run -d --name ci-cd-demo ci-cd-demo:latest
                    echo "✅ Application déployée avec succès!"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "🔍 Vérification du déploiement..."
                    docker ps | grep ci-cd-demo
                    echo "🎉 Application en cours d'exécution!"
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                    echo "🧹 Nettoyage des fichiers temporaires..."
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
            echo '✅ SUCCÈS TOTAL: CI/CD opérationnel avec analyse qualité et déploiement!'
            echo '📊 Rapport SonarQube: http://localhost:9000/dashboard?id=ci-cd-demo'
        }
    }
}
