pipeline {
    agent any
    environment {
        PATH = "/Users/deangdane/bin:/opt/homebrew/bin:${env.PATH}"
        COMPOSER_REPO = 'http://203.95.199.55:8081/repository/composer-proxy-i4/'
        NPM_REPO_GROUP = 'http://203.95.199.55:8081/repository/npm-group-i4/'
    }
    triggers {
        pollSCM('H/5 * * * *')  // Poll Git every 5 minutes
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Composer Install') {
            steps {
                sh '''
                /Users/deangdane/bin/composer config -g repo.packagist composer $COMPOSER_REPO
                /Users/deangdane/bin/composer install --no-interaction --prefer-dist --optimize-autoloader
                ls -la vendor/bin
                '''
            }
        }
        stage('NPM Install') {
            steps {
                sh '''
                /opt/homebrew/bin/npm config set registry $NPM_REPO_GROUP
                /opt/homebrew/bin/npm install
                '''
            }
        }
        stage('Run Tests') {
            steps {
                sh '''
                php ./vendor/bin/phpunit --configuration phpunit.xml
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh 'ansible-playbook -i localhost, web-deploy.yml'
            }
        }
    }
    post {
        failure {
            mail to: 'srengty@gmail.com, $GIT_COMMITTER_EMAIL',
                subject: "Build failed in Jenkins: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed. Please check Jenkins console output."
        }
    }
}
