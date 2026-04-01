pipeline {
    agent any

    environment {
        EC2_HOST = '44.212.73.120'
        EC2_USER = 'ubuntu'
        IMAGE_NAME = 'prakish980/petclinic:latest'
        CONTAINER_NAME = 'petclinic-app'
        APP_PORT = '8080'
    }

    stages {

        stage('Deploy Docker Container 1') {
            steps {
                echo '🚀 Deploying Docker container to EC2...'

                sshagent(['github-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '

                            # pull latest image
                            docker pull ${IMAGE_NAME} &&

                            # stop existing container (if running)
                            docker stop ${CONTAINER_NAME} || true &&

                            # remove old container
                             docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                            # run new container
                            docker run -d \
                                --name ${CONTAINER_NAME} \
                                -p 8081:8000 \
                                ${IMAGE_NAME}
                        '
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                echo '🩺 Checking app...'

                sshagent(['github-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            curl -I http://localhost:${APP_PORT} || echo "App not responding"
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Docker Deployment SUCCESS!'
        }
        failure {
            echo '❌ Deployment FAILED!'
        }
    }
}