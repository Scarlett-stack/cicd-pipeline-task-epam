pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
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
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker build -t nodedev:v1.0 .'
                    }
                }
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh '''
                            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodemain:v1.0 || true
                        '''
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh '''
                            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress nodedev:v1.0 || true
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker rm -f nodemain || true'
                        sh 'docker run -d --name nodemain --expose 3000 -p 3000:3000 nodemain:v1.0'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker rm -f nodedev || true'
                        sh 'docker run -d --name nodedev --expose 3001 -p 3001:3000 nodedev:v1.0'
                    }
                }
            }
        }
    }
}