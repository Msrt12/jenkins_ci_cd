pipeline {
    agent none
    
    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
                stash name: 'sourcecode', includes: '**/*'
            }
        }

        stage('Build') {
            agent {
                docker {
                     image 'node:20'   
                }
            }
            steps {
                unstash 'sourcecode'
                sh 'npm install'
                stash name: 'built', includes: '**/*'
            }
        }

        stage('Test') {
            agent {
                docker {
                     image 'node:20'   
                }
            }
            steps {
                unstash 'built'
                sh 'npm test || true'
                stash name: 'tested', includes: '**/*'
            }
        }

        stage('Docker build') {
            agent {
                docker {
                     image 'docker:dind'   
                }
            }
            steps {
                unstash 'tested'
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker build -t nodemain:v1.0 .'
                        stash name: 'builtmain', includes: '**/*'
                    }

                    if (env.BRANCH_NAME == 'dev') {
                        sh 'docker build -t nodedev:v1.0 .'
                        stash name: 'builtdev', includes: '**/*'
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                     image 'docker:dind'   
                }
            }
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        unstash 'builtmain'
                        sh 'docker rm -f nodemain || true'
                        sh 'docker run -d --name nodemain -p 3000:3000 nodemain:v1.0'
                    }

                    if (env.BRANCH_NAME == 'dev') {
                        unstash 'builtdev'
                        sh 'docker rm -f nodedev || true'
                        sh 'docker run -d --name nodedev -p 3001:3000 nodedev:v1.0'
                    }
                }
            }
        }
    }
}
