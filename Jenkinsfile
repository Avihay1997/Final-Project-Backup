pipeline {
    agent any

    environment {
        DOCKER_USER = "avihay1997"
        FLASK_CONTAINER_NAME = "flasksbackup-server"
        FLASK_IMAGE_NAME = "avihay1997/flaskbackup:latest"
        JENKINS_CONTAINER_NAME = "jenkinsbackup-server"
        JENKINS_IMAGE_NAME = "avihay1997/jenkinsbackup:latest"
        DOCKER_NETWORK = "jenkinsbackup-net"
        JENKINS_VOLUME = "jenkinsbackup-vol"
        EC2_PRIVATE_IP = "172.31.95.113"
        PEM_KEY = "/home/ubuntu/.ssh/private_key.pem"
        FLASK_PORT = "5001"
        JENKINS_PORT = "8081"
        JENKINS_AGENT_PORT = "50001"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git(url: 'https://github.com/Avihay1997/Final-Project-Backup', branch: 'main')
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t flaskbackup -f Dockerfile-flask ."
                sh "docker build -t jenkinsbackup -f Dockerfile-jenkins ."
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_TOKEN', variable: 'DOCKER_HUB_TOKEN')]) {
                    sh "echo DOCKER_HUB_TOKEN | docker login -u avihay1997 --password-stdin"
                    sh "docker push flaskbackup"
                    sh "docker push jenkinsbackup"
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i ${PEM_KEY} ubuntu@172.31.95.113 << EOF
                    docker network create jenkinsbackup-net || true
                    docker volume create jenkinsbackup-vol || true
                    
                    docker login -u avihay1997 --password-stdin
                    docker pull flaskbackup
                    docker pull jenkinsbackup
                    
                    docker ps -q --filter "name=flasksbackup-server" | grep -q . && docker stop flasksbackup-server && docker rm flasksbackup-server || true
                    docker ps -q --filter "name=jenkinsbackup-server" | grep -q . && docker stop jenkinsbackup-server && docker rm jenkinsbackup-server || true
                    
                    docker run -d --name שם לקוניטיינר של jenkins: --network jenkinsbackup-net -p 5001:5001 flaskbackup
                    docker run -d --name jenkinsbackup-server --network jenkinsbackup-net -p 8081:8081 -p 50001:50001 -v jenkinsbackup-vol:/var/jenkins_home jenkinsbackup
                    EOF
                    """
                }
            }
        }

        stage('Check Flask Health') {
            steps {
                script {
                    sh "curl --retry 5 --retry-delay 5 --retry-connrefused http://172.31.95.113:5001 || exit 1"
                }
            }
        }
    }
}
