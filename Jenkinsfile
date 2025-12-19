pipeline {
    agent any

    environment {
        IMAGE_NAME = 'notes-app:latest'
        CONTAINER_NAME = 'notes-app-container'
        PORT = '9092'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/BhaveshPrasad9402/notes-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "==== Building Docker Image ===="
                    docker build -t $IMAGE_NAME .
                '''
            }
        }
        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Docker_Hub_id_pwd',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "===Log in to Docker Hub==="
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo "===Tagging Image==="
                        docker tag $IMAGE_NAME $DOCKER_USER/$IMAGE_NAME
                        echo "===Passing Image to DockerHub ===="
                        docker push $DOCKER_USER/$IMAGE_NAME
                    
                    '''
                }
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    echo "==== Stopping old container ===="
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    echo "==== Running new container ===="
                    docker run -d \
                      --name $CONTAINER_NAME \
                      -p $PORT:80 \
                      -v notes-data:/data \
                      $IMAGE_NAME
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "==== Checking app response ===="
                    sleep 5
                    curl -s http://13.61.151.82:$PORT | head -n 20
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Notes app deployed successfully"
        }
        failure {
            echo "❌ Notes app deployment failed"
        }
    }
}
