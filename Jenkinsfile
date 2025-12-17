pipeline {
    agent any

    tools {
        nodejs 'nodejs'   // имя инструмента из Global Tool Configuration
    }


    stages {

        stage('Init env') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.ENV_NAME = 'prod'
                        env.APP_PORT = '3000'
                        env.IMAGE_NAME = 'nodemain:v1.0'
                        env.CONTAINER_NAME = 'node-main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.ENV_NAME = 'dev'
                        env.APP_PORT = '3001'
                        env.IMAGE_NAME = 'nodedev:v1.0'
                        env.CONTAINER_NAME = 'node-dev'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    echo """
                    =========================
                    Branch      : ${env.BRANCH_NAME}
                    Environment : ${env.ENV_NAME}
                    Port        : ${env.APP_PORT}
                    Image       : ${env.IMAGE_NAME}
                    Container   : ${env.CONTAINER_NAME}
                    =========================
                    """
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Build stage for ${env.ENV_NAME}"
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Test stage for ${env.ENV_NAME}"
                sh 'npm test || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${env.IMAGE_NAME}"
                sh "docker build -t ${env.IMAGE_NAME} ."
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${env.ENV_NAME} on port ${env.APP_PORT}"

                sh """
                  docker rm -f ${env.CONTAINER_NAME} || true

                  docker run -d \
                    --name ${env.CONTAINER_NAME} \
                    -p ${env.APP_PORT}:${env.APP_PORT} \
                    -e PORT=${env.APP_PORT} \
                    ${env.IMAGE_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful for ${env.ENV_NAME}"
        }
        failure {
            echo "Deployment failed for ${env.ENV_NAME}"
        }
        always {
            echo "Pipeline finished for branch ${env.BRANCH_NAME}"
        }
    }
}
