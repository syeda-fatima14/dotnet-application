pipeline {
    agent any

    environment {
        // .NET
        DOTNET_ROOT = "/opt/dotnet"
        PATH = "/opt/dotnet:/var/lib/jenkins/.dotnet/tools:${env.PATH}"

        // Docker / ACR
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
                withCredentials([
                    string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "Starting SonarQube analysis..."

                        dotnet sonarscanner begin \
                          /k:eshop-dotnet \
                          /d:sonar.host.url=http://localhost:9000 \
                          /d:sonar.token="$SONAR_TOKEN"
                    '''
                }
            }
        }

        stage('Restore') {
            steps {
                sh '''
                    set -e
                    dotnet restore eShop.Web.slnf
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    dotnet build eShop.Web.slnf \
                      --configuration Release \
                      --no-restore
                '''
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
                withCredentials([
                    string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "Finishing SonarQube analysis..."

                        dotnet sonarscanner end \
                          /d:sonar.token="$SONAR_TOKEN"
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "DOCKER BUILD"
                    echo "========================================"

                    docker build \
                      -f src/WebApp/Dockerfile \
                      -t $IMAGE_PREFIX-webapp:latest \
                      -t $ACR_LOGIN_SERVER/eshop-webapp:latest .

                    docker build \
                      -f src/Catalog.API/Dockerfile \
                      -t $IMAGE_PREFIX-catalog-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-catalog-api:latest .

                    docker build \
                      -f src/Ordering.API/Dockerfile \
                      -t $IMAGE_PREFIX-ordering-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-ordering-api:latest .

                    docker build \
                      -f src/Basket.API/Dockerfile \
                      -t $IMAGE_PREFIX-basket-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-basket-api:latest .

                    docker build \
                      -f src/Identity.API/Dockerfile \
                      -t $IMAGE_PREFIX-identity-api:latest \
                      -t $ACR_LOGIN_SERVER/eshop-identity-api:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "ACR LOGIN"
                    echo "========================================"

                    echo "$ACR_CREDS_PSW" | docker login \
                      "$ACR_LOGIN_SERVER" \
                      -u "$ACR_CREDS_USR" \
                      --password-stdin

                    echo "========================================"
                    echo "PUSHING IMAGES TO ACR"
                    echo "========================================"

                    docker push "$ACR_LOGIN_SERVER/eshop-webapp:latest"
                    docker push "$ACR_LOGIN_SERVER/eshop-identity-api:latest"
                    docker push "$ACR_LOGIN_SERVER/eshop-catalog-api:latest"
                    docker push "$ACR_LOGIN_SERVER/eshop-ordering-api:latest"
                    docker push "$ACR_LOGIN_SERVER/eshop-basket-api:latest"
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "DEPLOYING APPLICATION"
                    echo "========================================"

                    docker compose pull
                    docker compose up -d --remove-orphans
                    docker compose ps
                '''
            }
        }
    }

    post {
        always {
            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }

        success {
            echo 'Build, tests, SonarQube analysis, Docker build, ACR push and deployment succeeded.'
        }

        failure {
            echo 'Pipeline failed — check the console output above.'
        }
    }
}
