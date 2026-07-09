pipeline {
    agent any

    environment {
        // Your custom domain name
        APP_URL = 'http://aish-web-bank.com'
    }

    stages {
        stage('Pull Latest Code') {
            steps {
                checkout scm
            }
        }

        stage('Clean Environment') {
            steps {
                echo 'Stopping any previously running application instances...'
                // Fixed syntax: changed from docker-compose to docker compose
                sh 'docker compose down --remove-orphans'
            }
        }

        stage('Launch Martian Bank') {
            steps {
                echo 'Building and starting Martian Bank microservices via Docker...'
                // Fixed syntax: changed from docker-compose to docker compose
                sh 'docker compose up -d --build'
            }
        }

        stage('App Health Check') {
            steps {
                echo 'Waiting for the web application UI to become responsive...'
                timeout(time: 3, unit: 'MINUTES') {
                    sh '''
                        until $(curl --output /dev/null --silent --head --fail ${APP_URL}); do
                            printf '.'
                            sleep 5
                        done
                        echo "Martian Bank is fully up, stable, and running at ${APP_URL}!"
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment encountered an error. Gathering container logs...'
            sh 'docker compose logs --tail=50'
        }
    }
}