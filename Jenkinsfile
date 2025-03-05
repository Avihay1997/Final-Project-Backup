pipeline {
    agent any

    environment {
        DOCKER_USER = "avihay1997"
        FLASK_CONTAINER_NAME = "flask-app"
        FLASK_IMAGE_NAME = "avihay1997/flask-app:latest"
        EC2_PRIVATE_IP = "172.31.95.113"
        PEM_KEY = "/home/ubuntu/.ssh/private_key.pem"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git(url: 'https://github.com/Avihay1997/Project-Final', branch: 'main')
            }
        }

        stage('Build Flask Docker Image') {
            steps {
                sh "docker build -t ${FLASK_IMAGE_NAME} -f Dockerfile-flask ."
            }
        }

        stage('Push Flask Image to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_TOKEN', variable: 'DOCKER_HUB_TOKEN')]) {
                    sh "echo $DOCKER_HUB_TOKEN | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $FLASK_IMAGE_NAME"
                }
            }
        }

        stage('Deploy Flask on EC2') {
            steps {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i ${PEM_KEY} ubuntu@${EC2_PRIVATE_IP} << EOF
                    docker login -u $DOCKER_USER --password-stdin
                    docker pull ${FLASK_IMAGE_NAME}
                    docker ps -q --filter "name=${FLASK_CONTAINER_NAME}" | grep -q . && docker stop ${FLASK_CONTAINER_NAME} && docker rm ${FLASK_CONTAINER_NAME} || true
                    docker run -d --name ${FLASK_CONTAINER_NAME} -p 5000:5000 ${FLASK_IMAGE_NAME}
                    EOF
                    """
                }
            }
        }

        stage('Check Flask Health') {
            steps {
                script {
                    sh "curl --retry 5 --retry-delay 5 --retry-connrefused http://${EC2_PRIVATE_IP}:5000 || exit 1"
                }
            }
        }
    }
}
