pipeline {
    agent any
    environment {
        SONAR_HOST_URL = 'http://host.docker.internal:9000'
        SONAR_TOKEN    = 'squ_40246fa233e43a8d61b43d5dfbba79a400a3234b'
    }
    stages {
        stage('Check pytest') {
            steps {
                sh 'pytest --version'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }
        stage('SAST Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "sonar-scanner -Dsonar.projectKey=TP-Jenkins-Security -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN"
                }
            }
        }
        stage('SCA Scan') {
            steps {
                sh '''
                    # Créer le répertoire de données s'il n'existe pas
                    mkdir -p /var/jenkins_home/dependency-check-data
                    # Lancer l'analyse avec un répertoire de données spécifique
                    /opt/dependency-check/bin/dependency-check.sh --project "TP-Jenkins-Security" --scan . --format HTML --out dependency-check-report --data /var/jenkins_home/dependency-check-data
                '''
            }
        }
    }
    post {
        always {
            publishHTML([
                reportDir: 'dependency-check-report',
                reportFiles: 'dependency-check-report.html',
                reportName: 'OWASP Report',
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true
            ])
        }
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
}