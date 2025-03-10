pipeline {
    agent any
    environment {
        CONTAINER_NAME = "nginx_custom_lab" // Ім'я контейнера
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
            } // Додано автоматичне зупинення та видалення старих контейнерів перед новим розгортанням.
        }
        
        stage('Build nginx/custom') {
            steps {
                sh 'docker build -t nginx/custom:latest .'
            }
        }

        stage('Test nginx/custom') {
            steps {
                sh 'docker run --rm nginx/custom:latest nginx -t'        // Додано тестовий запуск контейнера.
                echo 'Container built and tested successfully!' // Змінено повідомлення про виконання
            }
        }

        stage('Deploy nginx/custom') {
            steps {
                sh 'docker run -d --name $CONTAINER_NAME -p 80:80 nginx/custom:latest'
                echo 'Deployment completed successfully!' // Повідомлення про результат виконання
            }
        }
    }
}
