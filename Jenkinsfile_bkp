pipeline {
    agent any

    environment {
        EC2_HOST = '44.212.73.120'           // ← Your EC2 public IP
        EC2_USER = 'ubuntu'
        APP_DIR  = '/home/ubuntu/myapp/Jenkins_l1'
        REPO_URL = 'https://github.com/kishorekcr/Jenkins_l1'
        DOCKERHUB_USER = 'prakish980'
        IMAGE_NAME    = 'myapp'
        IMAGE_TAG     = "v${BUILD_NUMBER}"  
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Pulling code from GitHub...'
                git branch: 'main',
                    url: "${REPO_URL}",
                    credentialsId: 'github-cred'
    }
}

        stage('Docker Build') {
            steps {
                echo '🐳 Building Docker image...'
                sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} \
                               ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Docker Push') {
            steps {
                echo '📦 Pushing image to DockerHub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo '🚀 Deploying on EC2...'
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:latest &&
                            docker stop myapp || true &&
                            docker rm myapp || true &&
                            docker run -d \
                                --name myapp \
                                -p 3000:3000 \
                                --restart always \
                                ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                        '
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo '🚀 Deploying to EC2...'
                sshagent(['github-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            if [ ! -d "${APP_DIR}/.git" ]; then
                                git clone ${REPO_URL} ${APP_DIR};
                            fi &&
                            cd ${APP_DIR} &&
                            git pull origin main &&
                            npm install --production &&
                            command -v pm2 >/dev/null 2>&1 || sudo npm install -g pm2 &&
                            pm2 restart myapp || pm2 start ${APP_DIR}/app.js --name myapp
                        '
                    """
        }
    }
}

        stage('Health Check') {
            steps {
                echo '🩺 Checking if app is live...'
                sshagent(['github-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            curl -s http://localhost:3000 || echo "App not responding!"
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment SUCCESS!'
        }
        failure {
            echo '❌ Deployment FAILED! Check logs.'
        }
    }
}