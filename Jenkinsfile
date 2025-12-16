pipeline {
    agent any

    environment {
        VENV_DIR = "venv"
    }

    tools {
        sonarScanner 'sonar-scanner'
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
                    python3 -m venv ${VENV_DIR}
                    ${VENV_DIR}/bin/python -m ensurepip --upgrade
                    ${VENV_DIR}/bin/python -m pip install --upgrade "pip<24.1"
                    ${VENV_DIR}/bin/python -m pip install -r requirements.txt
                    ${VENV_DIR}/bin/python -m pip install -e .
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    ${VENV_DIR}/bin/python -m pytest
                '''
            }
        }

        stage('SonarQube SAST Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=simple-cicd-app \
                          -Dsonar.projectName=simple-cicd-app \
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

