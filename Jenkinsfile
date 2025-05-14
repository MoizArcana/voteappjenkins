pipeline {
    agent any

    environment {
        IMAGE_NAME = "moiz3388/voteappjenkins"
        TAG = "latest"
        EC2_HOST = "ubuntu@54.89.241.89" 
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/MoizArcana/voteappjenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${IMAGE_NAME}:${TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ec2-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${EC2_HOST} << 'EOF'
                        docker pull ${IMAGE_NAME}:${TAG}
                        docker stop voteapp || true
                        docker rm voteapp || true
                        docker run -d --name voteapp -p 80:80 ${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed and deployed to EC2 successfully.'
        }
        failure {
            echo 'There was an issue with the pipeline process.'
        }
    }
}
