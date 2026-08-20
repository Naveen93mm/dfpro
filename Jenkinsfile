pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveen93mm/novaloc93"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DEPLOY_SERVER = "3.110.144.109"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Naveen93mm/dfpro.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent(['vm']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOY_SERVER} '
                        sudo docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sudo docker stop novaloc993 || true
                        sudo docker rm novaloc993 || true
                        sudo docker run -d -p 90:80 --name novaloc993 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful "
        }
        failure {
            echo "Pipeline Failed check Console Output"
        }
    }
}
