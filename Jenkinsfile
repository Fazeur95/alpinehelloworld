@Library('shared-library')_

pipeline {
    environment {
        ID_DOCKER = "${ID_DOCKER_PARAMS}"
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "${ID_DOCKER}-staging"
        PRODUCTION = "${ID_DOCKER}-production"
    }
    agent any
    stages {
        stage('Build image') {
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Run container based on builded image') {
            steps {
                script {
                    sh '''
                        echo "Clean Environment"
                        docker rm -f $IMAGE_NAME || echo "container does not exist"
                        docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                    '''
                }
            }
        }
        stage('Test image') {
            steps {
                script {
                    sh '''
                        curl -u faouizi.mzebla@ynov.com:${DOCKERHUB_PASSWORD_PSW} http://172.17.0.1:${PORT_EXPOSED}
                    '''
                }
            }
        }
        stage('Clean Container') {
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''
                }
            }
        }
        stage ('Login and Push Image on docker hub') {
            environment {
                DOCKERHUB_PASSWORD = credentials('dockerhub')
            }            
            steps {
                script {
                    sh '''
                        echo $DOCKERHUB_PASSWORD | docker login -u $ID_DOCKER --password-stdin
                        docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Install Node.js and Heroku CLI') {
            steps {
                script {
                    sh '''
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
                        export NVM_DIR="$HOME/.nvm"
                        [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                        nvm install 14
                        nvm use 14
                        npm install -g heroku@8.11.0
                    '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }  
            steps {
                script {
                    sh '''
                        export PATH="$HOME/.nvm/versions/node/v14.21.3/bin:$PATH"
                        heroku container:login
                        heroku create $STAGING || echo "project already exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''
                }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/production' }
            }
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }  
            steps {
                script {
                    sh '''
                        export PATH="$HOME/.nvm/versions/node/v14.21.3/bin:$PATH"
                        heroku container:login
                        heroku create $PRODUCTION || echo "project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }
    }
    post {
        always {
            script {
                slackNotifier(currentBuild.result)
            }
        }  
    }
}
