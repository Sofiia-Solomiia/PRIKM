pipeline {
    agent any
    environment {
        CONTAINER_NAME = "nginx_lab"
    }
    
    stages {
        stage('Start') {
            steps {
                echo 'Lab_1: nginx/custom'
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
        
        stage('Build nginx/custom') {
            steps {
                sh 'docker build -t nginx/custom:latest .'
            }
        }

        stage('Test nginx/custom') {
            steps {
                sh 'docker run --rm nginx/custom:latest nginx -t' 
                echo 'Container built and tested successfully!' 
            }
        }

        stage('Deploy nginx/custom') {
            steps {
                sh 'docker run -d --name $CONTAINER_NAME -p 80:80 nginx/custom:latest'
                echo 'Deployment completed successfully!' 
            }
        }
    }
}
