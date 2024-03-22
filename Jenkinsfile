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
                    docker.build("${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG")
                }
            }
        }
        stage('Run container based on builded image') {
            steps {
                script {
                    docker.withRegistry('','dockerhub') {
                        dockerImage = docker.image("${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG")
                        dockerImage.run("-p ${PORT_EXPOSED}:5000 -e PORT=5000 --name $IMAGE_NAME")
                    }
                }
            }
        }
        stage('Test image') {
            steps {
                script {
                    sh "curl -u faouizi.mzebla@ynov.com:Louxor95100 http://172.17.0.1:${PORT_EXPOSED}"
                }
            }
        }
        stage('Clean Container') {
            steps {
                script {
                    docker.stop("$IMAGE_NAME")
                    docker.removeContainer("$IMAGE_NAME")
                }
            }
        }
        stage ('Login and Push Image on docker hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_USERNAME, DOCKERHUB_PASSWORD) {
                            docker.image("${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG").push()
                        }
                    }
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
        stage('Deploy to Heroku') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            steps {
                script {
                    sh '''
                        export HEROKU_API_KEY=$(echo "$HEROKU_API_KEY" | base64 -d)
                        heroku container:login
                        heroku create $STAGING || echo "Project already exists"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''
                }
            }
        }
    }
}
