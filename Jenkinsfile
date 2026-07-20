pipeline {
    agent any

    tools {
        nodejs 'node-16'
    }

    environment {
        IMAGE_NAME = "cicd-pipeline-app"
        DOCKERHUB_USER = "erosleal"
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
                    def branch = env.BRANCH_NAME ?: 'main'
                    env.APP_BRANCH = branch
                    if (branch == 'main') {
                        env.PORT = '3000'
                    } else if (branch == 'dev') {
                        env.PORT = '3001'
                    } else {
                        env.PORT = '3000'
                    }
                    echo "Branch: ${env.APP_BRANCH} -> Port: ${env.PORT}"
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
                sh "docker build -t ${IMAGE_NAME}:${env.APP_BRANCH} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker tag ${IMAGE_NAME}:${env.APP_BRANCH} ${DOCKERHUB_USER}/${IMAGE_NAME}:${env.APP_BRANCH}"
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${env.APP_BRANCH}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerName = "${IMAGE_NAME}-${env.APP_BRANCH}"
                    sh "docker rm -f ${containerName} || true"
                    sh "docker run -d --name ${containerName} -p ${env.PORT}:3000 ${IMAGE_NAME}:${env.APP_BRANCH}"
                }
            }
        }

        stage('Trigger deploy pipeline') {
            steps {
                script {
                    if (env.APP_BRANCH == 'main') {
                        build job: 'Deploy_to_main', wait: false
                    } else if (env.APP_BRANCH == 'dev') {
                        build job: 'Deploy_to_dev', wait: false
                    }
                }
            }
        }
    }
}
