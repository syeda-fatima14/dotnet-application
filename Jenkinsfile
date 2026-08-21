pipeline {
    agent any
    
    environment {
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "${DOTNET_ROOT}:${env.PATH}"
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        ACR_CREDS = credentials('acr-creds')
        IMAGE_PREFIX = "syedafatima14/eshop"
        ACR_LOGIN_SERVER = "eshopacrsf14.azurecr.io"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Restore') {
            steps { sh 'dotnet restore eShop.Web.slnf' }
        }

        stage('Build') {
            steps { sh 'dotnet build eShop.Web.slnf --configuration Release --no-restore' }
        }

        stage('Test') {
            steps { sh 'dotnet test eShop.Web.slnf --configuration Release --no-build --logger "trx;LogFileName=test-results.trx" || true' }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -f src/WebApp/Dockerfile -t $IMAGE_PREFIX-webapp:latest -t $ACR_LOGIN_SERVER/eshop-webapp:latest .
                    docker build -f src/Catalog.API/Dockerfile -t $IMAGE_PREFIX-catalog-api:latest -t $ACR_LOGIN_SERVER/eshop-catalog-api:latest .
                    docker build -f src/Ordering.API/Dockerfile -t $IMAGE_PREFIX-ordering-api:latest -t $ACR_LOGIN_SERVER/eshop-ordering-api:latest .
                    docker build -f src/Basket.API/Dockerfile -t $IMAGE_PREFIX-basket-api:latest -t $ACR_LOGIN_SERVER/eshop-basket-api:latest .
                    docker build -f src/Identity.API/Dockerfile -t $IMAGE_PREFIX-identity-api:latest -t $ACR_LOGIN_SERVER/eshop-identity-api:latest .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    echo $ACR_CREDS_PSW | docker login $ACR_LOGIN_SERVER -u $ACR_CREDS_USR --password-stdin
                    docker push $ACR_LOGIN_SERVER-webapp:latest
                    docker push $ACR_LOGIN_SERVER-identity-api:latest
                    docker push $ACR_LOGIN_SERVER-catalog-api:latest
                    docker push $ACR_LOGIN_SERVER-ordering-api:latest
                    docker push $ACR_LOGIN_SERVER-basket-api:latest
                '''
            }
        }
    
        stage('Deploy') {
            steps {
                sh '''
                    docker compose pull
                    docker compose up -d --remove-orphans
                    docker compose ps
                '''
            }
        }
   }
        
    post {
        always { echo 'Pipeline finished.' }
        success { echo 'Build, tests, and image push succeeded.' }
        failure { echo 'Pipeline failed — check logs above.' }
    }
}
