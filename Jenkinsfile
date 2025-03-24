pipeline {
    agent any
    environment {
        CONTAINER_NAME = "custom__lab2" // Ім'я контейнера
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
                    echo "No existing container found, skipping cleanup"
                fi
                '''
            } // Додано автоматичне зупинення та видалення старих контейнерів перед новим розгортанням.
        }
        
        stage('Image build') {
            steps {
                sh "docker build -t prikm:latest ."
                sh "docker tag prikm sofiiasolomiia/prikm:latest"
                sh "docker tag prikm sofiiasolomiia/prikm:$BUILD_NUMBER"
            }
        }
        stage('Push to registry') {
            steps {
                withDockerRegistry([ credentialsId: "docker-hub-credentials", url: "" ])
                {
                    sh "docker push sofiiasolomiia/prikm:latest"
                    sh "docker push sofiiasolomiia/prikm:$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy image'){
            steps{
                sh "docker run -d -p 80:80 sofiiasolomiia/prikm"
            }
        }
    }
}
