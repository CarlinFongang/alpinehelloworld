pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "Carlinfg-staging"
        PRODUCTION = "Carlinfg-production"
        IP_ADDRESS = "44.222.148.16"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t Carlinfg/$IMAGE_NAME:$IMAGE_TAG .'  
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 Carlinfg/$IMAGE_NAME:$IMAGE_TAG
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
                        curl -I http://IP_ADDRESS:80 | grep -i "Hello World!"
                    '''
                    
                }
            }
        }
        stage('Clean test') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME && docker rm $IMAGE_NAME
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
                HEROKU_API_KEY = credentials('HEROKU_API_KEY')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $TAGING || echo "project already exist"
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
                HEROKU_API_KEY = credentials('HEROKU_API_KEY')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $TAGING || echo "project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web 
                    '''
                }
            }
        }
    }
}
