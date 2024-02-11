/* import shared library */
@Library('carlinfg-shared-library')_

pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "carlinfg-staging"
        PRODUCTION = "carlinfg-production"
        IP_ADDRESS = "44.197.120.217"
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
                        curl -I "http://$IP_ADDRESS:80"
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
        stage('Destroy Staging and Production Deployments') {
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
                        sleep 900
                        heroku apps:destroy --app $STAGING --confirm $STAGING
                        heroku apps:destroy --app $PRODUCTION --confirm $PRODUCTION
                        '''
                    }
                }
            }
    }
    post {
        always {
            script {
                slackNotifier currentBuild.result
      }
    }  
  }

}
