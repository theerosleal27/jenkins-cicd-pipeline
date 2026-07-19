pipeline {
    agent any

    tools {
        nodejs 'node-16'
    }

    environment {
        IMAGE_NAME = "cicd-pipeline-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set environment variables') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.PORT = '3001'
                    } else {
                        env.PORT = '3000'
                    }
                    echo "Branch: ${env.BRANCH_NAME} -> Port: ${env.PORT}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x scripts/build.sh'
                sh './scripts/build.sh'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${env.BRANCH_NAME} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerName = "${IMAGE_NAME}-${env.BRANCH_NAME}"
                    sh "docker rm -f ${containerName} || true"
                    sh "docker run -d --name ${containerName} -p ${env.PORT}:3000 ${IMAGE_NAME}:${env.BRANCH_NAME}"
                }
            }
        }
    }
}
