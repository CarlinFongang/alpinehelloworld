/* import shared library */
@Library('carlinfg-shared-library')_

pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "carlinfg-staging"
        PRODUCTION = "carlinfg-production"
        IP_ADDRESS = "44.197.243.31"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t carlfg/$IMAGE_NAME:$IMAGE_TAG .'  
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 carlfg/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        curl -I "http://$IP_ADDRESS"
                    '''  
                }
            }
        }
        stage('Clean test') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME 
                        docker rm $IMAGE_NAME
                    '''
                    
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                        npm i -g heroku@7.68.0
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
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                        npm i -g heroku@7.68.0
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
                SlackNotifier currentBuild.result
      }
    }  
  } 
}
