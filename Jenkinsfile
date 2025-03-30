properties([
    pipelineTriggers([]),
    office365ConnectorWebhooks([
        [
            name: 'Teams-O365',
            url: 'https://lpnu.webhook.office.com/webhookb2/8f322f9f-54a7-4daf-9f1a-da81939d84af@7631cd62-5187-4e15-8b8e-ef653e366e7a/IncomingWebhook/2eef2504c44b45b2992456f173a3f49a/60fa48bd-5fc5-49ab-b31e-390ca5651e30/V235YDb95QIp274jIQm2nRveb1ZIk-_AUSrNsuundo0mI1',
            startNotification: false,
            notifySuccess: true,
            notifyAborted: false,
            notifyNotBuilt: false,
            notifyUnstable: true,
            notifyFailure: true,
            notifyBackToNormal: true,
            notifyRepeatedFailure: false,
            timeout: 30000
        ]
    ])
])

pipeline {
    agent any
    environment {
        CONTAINER_NAME = "custom__lab2" // Ім'я контейнера
        TEAMS_WEBHOOK_URL = "https://lpnu.webhook.office.com/webhookb2/8f322f9f-54a7-4daf-9f1a-da81939d84af@7631cd62-5187-4e15-8b8e-ef653e366e7a/IncomingWebhook/2eef2504c44b45b2992456f173a3f49a/60fa48bd-5fc5-49ab-b31e-390ca5651e30/V235YDb95QIp274jIQm2nRveb1ZIk-_AUSrNsuundo0mI1"
    }
    
    stages {
        stage('Start') {
            steps {
                echo 'Lab_2: started by GitHub'
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

        stage('Image build') {
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
                withDockerRegistry([credentialsId: "dockerhub_token", url: ""]) {
                    sh '''
                    docker push $IMAGE_NAME:latest
                    docker push $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy image') {
            steps {
                sh '''
                docker run -d --name $CONTAINER_NAME -p 80:80 $IMAGE_NAME:latest
                echo "Deployment completed successfully!"
                '''
            }
        }
    }
    post {
        success {
            office365ConnectorSend message: "Build and deployment successful for tag: latest",
                webhookUrl: env.TEAMS_WEBHOOK_URL
        }
        failure {
            office365ConnectorSend message: "Build failed! Check Jenkins logs.",
                webhookUrl: env.TEAMS_WEBHOOK_URL
        }
    }
}
