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

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    venv/bin/python -m pip install --upgrade pip
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
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=simple-cicd-app \
                        -Dsonar.projectName=simple-cicd-app \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=app \
                        -Dsonar.tests=tests \
                        -Dsonar.python.version=3.10
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI + SAST pipeline succeeded'
        }
        failure {
            echo '❌ CI or SAST failed'
        }
    }
}

