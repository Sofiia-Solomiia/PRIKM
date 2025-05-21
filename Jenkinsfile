properties([
    parameters([
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Tag for Docker image')
    ]),
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
 
        TEAMS_WEBHOOK_URL = "https://lpnu.webhook.office.com/webhookb2/8f322f9f-54a7-4daf-9f1a-da81939d84af@7631cd62-5187-4e15-8b8e-ef653e366e7a/IncomingWebhook/2eef2504c44b45b2992456f173a3f49a/60fa48bd-5fc5-49ab-b31e-390ca5651e30/V235YDb95QIp274jIQm2nRveb1ZIk-_AUSrNsuundo0mI1"
    }
    
    stages {
        stage('Start') {
            steps {
                echo "Lab_2: started by GitHub"
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

        stage('Image build') {
            steps {
                sh "docker build -t prikm:$IMAGE_TAG ."
                sh "docker tag prikm:$IMAGE_TAG sofiiasolomiia/prikm:$IMAGE_TAG"
                sh "docker tag prikm:$IMAGE_TAG sofiiasolomiia/prikm:$BUILD_NUMBER"
            }
        }
        
        stage('Push to registry') {
            steps {
                withDockerRegistry([ credentialsId: "docker_lab_token", url: "" ]) {
                    sh "docker push sofiiasolomiia/prikm:$IMAGE_TAG"
                    sh "docker push sofiiasolomiia/prikm:$BUILD_NUMBER"
                }
            }
        }
        
        stage('Deploy image') {
            steps {
                sh "docker run -d -p 8881:80 --name custom_lab3 sofiiasolomiia/prikm:$IMAGE_TAG"
            }
        }
    }

    post {
        success {
           withCredentials([vaultString(credentialsId: 'vault-secret-text', variable: 'TEAMS_WEBHOOK_URL')]) {
            sh "curl -X POST -H "Content-Type: application/json" \
                -d '{"text": "✅ Build and deployment successful for tag: '''"$IMAGE_TAG"'''"}' \
                $TEAMS_WEBHOOK_URL"
            }
        }
        failure {
            withCredentials([vaultString(credentialsId: 'vault-teams-webhook', variable: 'TEAMS_WEBHOOK_URL')]) {
            sh "curl -X POST -H "Content-Type: application/json" \
                -d '{"text": "❌ Build failed! Check Jenkins logs."}' \
                $TEAMS_WEBHOOK_URL"
        }
        }
    }
}
