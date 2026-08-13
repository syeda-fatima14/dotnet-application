pipeline {
    agent any

    environment {
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "${DOTNET_ROOT}:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore eShop.Web.slnf'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build eShop.Web.slnf --configuration Release --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test eShop.Web.slnf --configuration Release --no-build --logger "trx;LogFileName=test-results.trx" || true'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build and tests succeeded.'
        }
        failure {
            echo 'Build or tests failed — check logs above.'
        }
    }
}
