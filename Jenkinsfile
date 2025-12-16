pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Env') {
            steps {
                sh '''
                    python3 -m venv venv
                    venv/bin/python -m pip install --upgrade pip
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
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=simple-cicd-app \
                        -Dsonar.projectName=simple-cicd-app \
                        -Dsonar.sources=app \
                        -Dsonar.python.version=3.10
                    '''
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
    }

    post {
        success {
            echo '✅ CI + SAST + Quality Gate PASSED'
        }
        failure {
            echo '❌ Quality Gate FAILED — Fix security issues'
        }
    }
}

