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
                    rm -rf venv

                    python3 -m venv venv

                    venv/bin/python -m pip install --no-cache-dir pip==23.2.1
                    venv/bin/python -m pip install --no-cache-dir -r requirements.txt
                    venv/bin/python -m pip install --no-cache-dir -e .
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

