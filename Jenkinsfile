pipeline {
    agent any
    environment {
        SONAR_HOST_URL = 'http://host.docker.internal:9000'   // ou 172.17.0.1
        SONAR_TOKEN    = 'squ_40246fa233e43a8d61b43d5dfbba79a400a3234b'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/hasnaBourhym/tp-jenkins-security.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt'
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
                    sh 'sonar-scanner -Dsonar.projectKey=TP-Jenkins-Security -Dsonar.sources=.'
                }
            }
        }
        stage('SCA Scan') {
            steps {
                dependencyCheckAnalyzer(
                    datadir: 'dependency-check-data',
                    pattern: '**/*.jar,**/*.war,**/*.py',
                    scanpath: '.',
                    outpath: 'dependency-check-report',
                    failBuildOnCVSS: '7.0',
                    toolId: 'DP_CHECK'   // <-- Ajouté pour utiliser l'outil configuré
                )
                dependencyCheckPublisher pattern: 'dependency-check-report/dependency-check-report.xml'
            }
        }
    }
    post {
        always {
            publishHTML([
                reportDir: 'dependency-check-report',
                reportFiles: 'dependency-check-report.html',
                reportName: 'OWASP Report'
            ])
        }
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
}