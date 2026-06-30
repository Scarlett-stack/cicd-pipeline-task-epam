pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        DOCKERHUB_REPO = 'darikb123/cicd-pipeline-task'
        DOCKERHUB_CREDS = 'bdbcb634-e268-4196-b923-bd2e303a25c6'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Hadolint Dockerfile') {
            steps {
                sh 'docker run --rm -i hadolint/hadolint < Dockerfile || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker build -t nodemain:v1.0 .'
                        sh "docker tag nodemain:v1.0 ${DOCKERHUB_REPO}:nodemain-v1.0"
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker build -t nodedev:v1.0 .'
                        sh "docker tag nodedev:v1.0 ${DOCKERHUB_REPO}:nodedev-v1.0"
                    }
                }
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodemain:v1.0 || true'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodedev:v1.0 || true'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'

                        if (env.BRANCH_NAME == 'main') {
                            sh "docker push ${DOCKERHUB_REPO}:nodemain-v1.0"
                        } else if (env.BRANCH_NAME == 'dev') {
                            sh "docker push ${DOCKERHUB_REPO}:nodedev-v1.0"
                        }
                    }
                }
            }
        }

        stage('Trigger Deploy Pipeline') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main', wait: false
                    } else if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev', wait: false
                    }
                }
            }
        }
    }
}