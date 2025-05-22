pipeline {
    agent any
   
    environment {
        CONTAINER_NAME = "prikm_lab2"
        IMAGE_NAME = "sofiiasolomiia/prikm"
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
                if [ "$(docker ps -aq -f name=$CONTAINER_NAME)" ]; then
                    echo "Stopping and removing existing container: $CONTAINER_NAME"
                    docker stop $CONTAINER_NAME && docker rm $CONTAINER_NAME
                else
                    echo "No existing container found, skipping cleanup."
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
                sh 'hcp profile set vault-secrets/app Lab-7'
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
                env.webhookUrl = sh(script: 'hcp vault-secrets secrets open teams_microsoft_webhook --format=json | jq -r .static_version.value', returnStdout: true).trim()
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
