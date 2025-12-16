pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'SonarQube'   // Jenkins → Manage Jenkins → Configure System
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

                    # FIX: ensure stable pip (avoid pip 25.x crash)
                    venv/bin/python -m ensurepip --upgrade
                    venv/bin/python -m pip install "pip<24.1"

                    venv/bin/pip install -r requirements.txt
                    venv/bin/pip install -e .
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    venv/bin/pytest
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

