pipeline {
    agent any

    environment {
        CONTAINER_NAME = "prikm_lab2"
        IMAGE_NAME = "sofiiasolomiia/prikm"
        TEAMS_WEBHOOK_URL = "https://lpnu.webhook.office.com/webhookb2/8f322f9f-54a7-4daf-9f1a-d..." // ← допиши повний
    }

    stages {
        stage('🔰 Початок процесу') {
            steps {
                echo 'Старт: Lab_7 pipeline'
            }
        }

        stage('Cleanup old containers') {
            steps {
                sh '''
                if [ "$(docker ps -aq -f name=custom_lab3)" ]; then
                    echo "Stopping and removing existing container: custom_lab3"
                    docker stop custom_lab3 && docker rm custom_lab3
                else
                    echo "No existing container found, skipping cleanup"
                fi
                '''
            }
        }

        stage('🔐 Аутентифікація до HCP') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'hcp_credentials',
                    usernameVariable: 'HCP_CLIENT_ID',
                    passwordVariable: 'HCP_CLIENT_SECRET'
                )]) {
                    script {
                        sh 'hcp auth login --client-id $HCP_CLIENT_ID --client-secret $HCP_CLIENT_SECRET'
                    }
                }
            }
        }

        stage('⚙️ Ініціалізація HCP профілю') {
            steps {
                sh 'hcp profile set vault-secrets/app vault-server-app-pavlyshyn'
            }
        }

        stage('🐳 Збірка Docker образу nginx/custom') {
            steps {
                sh '''
                docker build -t prikm:latest .
                docker tag prikm $IMAGE_NAME:latest
                docker tag prikm $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Push to registry') {
            steps {
                withDockerRegistry([credentialsId: "docker_lab_token", url: ""]) {
                    sh '''
                    docker push $IMAGE_NAME:latest
                    docker push $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('🚀 Деплой nginx/custom контейнера') {
            steps {
                sh '''
                docker run -d --name $CONTAINER_NAME -p 81:80 $IMAGE_NAME:latest
                echo "Deployment completed successfully!"
                '''
            }
        }

        stage('✅ Завершення процесу') {
            steps {
                echo 'Завершення: Lab_7 pipeline'
            }
        }
    }

    post {
        always {
            script {
                env.webhookUrl = sh(script: 'hcp vault-secrets secrets open teams_microsoft_webhook --field=url', returnStdout: true).trim()
            }
        }

        success {
            office365ConnectorSend(
                webhookUrl: webhookUrl,
                message: "✅ Збірка пройшла успішно!",
                status: "Success",
                color: "00FF00"
            )
        }

        failure {
            office365ConnectorSend(
                webhookUrl: webhookUrl,
                message: "❌ Збірка зазнала невдачі!",
                status: "Failure",
                color: "FF0000"
            )
        }
    }
}
