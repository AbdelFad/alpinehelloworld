pipeline{
    environment{
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "fadaaz-staging"
        PRODUCTION = "fadaaz-production"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t fadaaz/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Run container based on build image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run -d --name ${IMAGE_NAME} -p 80:5000 -e PORT=5000 fadaaz/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 3600
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Hello world!"
                    '''
                }
            }
        }
        stage('Clean Container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop ${IMAGE_NAME} 
                        docker rm ${IMAGE_NAME} -f
                    '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression {GIT_BRANCH == 'origine/master' }
            }
            agent any
            environment {
                DIGITALOCEAN_API_KEY = credentials('alpinehelloworld_key')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $STAGING || echo "projet already exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''
                }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                expression {GIT_BRANCH == 'origine/master' }
            }
            agent any
            environment {
               DIGITALOCEAN_API_KEY = credentials('alpinehelloworld_key')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "projet already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }
    }
}
