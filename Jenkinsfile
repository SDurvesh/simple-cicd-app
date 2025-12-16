pipeline {
    agent any

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
    }

    post {
        success {
            echo '✅ CI Pipeline completed successfully'
        }
        failure {
            echo '❌ CI Pipeline failed'
        }
    }
}

