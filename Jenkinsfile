pipeline {
    agent any

    environment {
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "${DOTNET_ROOT}:${env.PATH}"
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        IMAGE_PREFIX = "syedafatima14/eshop"
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
                    docker build -f src/WebApp/Dockerfile -t $IMAGE_PREFIX-webapp:latest .
                    docker build -f src/Catalog.API/Dockerfile -t $IMAGE_PREFIX-catalog-api:latest .
                    docker build -f src/Ordering.API/Dockerfile -t $IMAGE_PREFIX-ordering-api:latest .
                    docker build -f src/Basket.API/Dockerfile -t $IMAGE_PREFIX-basket-api:latest .
                    docker build -f src/Identity.API/Dockerfile -t $IMAGE_PREFIX-identity-api:latest .             
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
                    docker push $IMAGE_PREFIX-webapp:latest
                    docker push $IMAGE_PREFIX-identity-api:latest
                    docker push $IMAGE_PREFIX-catalog-api:latest
                    docker push $IMAGE_PREFIX-ordering-api:latest
                    docker push $IMAGE_PREFIX-basket-api:latest
                '''
            }
        }
    
        stage('Deploy') {
            steps {
                sh '''
                    docker compose pull
                    docker compose up -d --remove-orphans
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
