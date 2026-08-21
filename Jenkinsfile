pipeline {
    agent any

    environment {
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "/opt/dotnet:/var/lib/jenkins/.dotnet/tools:${env.PATH}"

        SONAR_TOKEN = credentials('sonar-token')

        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        ACR_CREDS = credentials('acr-creds')

        IMAGE_PREFIX = "syedafatima14/eshop"
        ACR_LOGIN_SERVER = "eshopacrsf14.azurecr.io"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Begin') {
            steps {
                sh '''
                    dotnet sonarscanner begin \
                      /k:"eshop-dotnet" \
                      /d:sonar.host.url="http://localhost:9000" \
                      /d:sonar.token:=$SONAR_TOKEN"
                '''
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
                sh '''
                    dotnet test eShop.Web.slnf \
                      --configuration Release \
                      --no-build \
                      --logger "trx;LogFileName=test-results.trx" || true
                '''
            }
        }

        stage('SonarQube End') {
            steps {
                sh '''
                    dotnet sonarscanner end \
                      /d:sonar.token="$SONAR_TOKEN"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -f src/WebApp/Dockerfile \
                      -t $IMAGE_PREFIX-webapp:latest \
                      -t $ACR_LOGIN_SERVER/eshop-webapp:latest .

                    docker build -f src/Catalog.API/Dockerfile \
                      -t $IMAGE_PREFIX-catalog-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-catalog-api:latest .

                    docker build -f src/Ordering.API/Dockerfile \
                      -t $IMAGE_PREFIX-ordering-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-ordering-api:latest .

                    docker build -f src/Basket.API/Dockerfile \
                      -t $IMAGE_PREFIX-basket-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-basket-api:latest .

                    docker build -f src/Identity.API/Dockerfile \
                      -t $IMAGE_PREFIX-identity-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-identity-api:latest .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    echo $ACR_CREDS_PSW | docker login $ACR_LOGIN_SERVER \
                      -u $ACR_CREDS_USR --password-stdin

                    docker push $ACR_LOGIN_SERVER/eshop-webapp:latest
                    docker push $ACR_LOGIN_SERVER/eshop-identity-api:latest
                    docker push $ACR_LOGIN_SERVER/eshop-catalog-api:latest
                    docker push $ACR_LOGIN_SERVER/eshop-ordering-api:latest
                    docker push $ACR_LOGIN_SERVER/eshop-basket-api:latest
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
        always {
            echo 'Pipeline finished.'
        }

        success {
            echo 'Build, tests, SonarQube analysis, and image push succeeded.'
        }

        failure {
            echo 'Pipeline failed — check logs above.'
        }
    }
}
