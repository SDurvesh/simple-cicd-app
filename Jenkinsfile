pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv

                    # Always use python -m pip (never pip directly)
                    venv/bin/python -m ensurepip --upgrade
                    venv/bin/python -m pip install --upgrade "pip<24.1"

                    venv/bin/python -m pip install -r requirements.txt
                    venv/bin/python -m pip install -e .
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    venv/bin/python -m pytest
                '''
            }
        }

        stage('SonarQube SAST Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=simple-cicd-app \
                          -Dsonar.projectName=simple-cicd-app \
                          -Dsonar.projectVersion=1.0 \
                          -Dsonar.sources=app \
                          -Dsonar.tests=tests \
                          -Dsonar.python.version=3.10 \
                          -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI + SAST + Quality Gate PASSED'
        }
        failure {
            echo '❌ CI or Quality Gate FAILED — fix issues before merge'
        }
    }
}

