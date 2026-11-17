pipeline {
    agent any

    environment {
        IMAGE_NAME = 'flask-test'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

         stage('Tests') {
             steps {
                 sh 'docker build -f Dockerfile.test -t $IMAGE_NAME-tests:$BUILD_NUMBER .'
                 sh 'docker run --rm $IMAGE_NAME-tests:$BUILD_NUMBER'
            }
        }

        stage('Build image') {
            steps {
                sh 'docker build --no-cache --pull -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Scan config (Trivy)') {
            steps {
                sh 'mkdir -p reports'
                sh 'trivy config --format json --output reports/trivy-config.json .'
                sh 'trivy config --severity HIGH,CRITICAL --exit-code 1 .'
            }
        }

        stage('Scan image (Trivy)') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL --ignore-unfixed --format json --output reports/trivy-image.json $IMAGE_NAME:$BUILD_NUMBER'
                sh 'trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 $IMAGE_NAME:$BUILD_NUMBER'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/*.json', allowEmptyArchive: true
        }
    }
}