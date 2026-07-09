pipeline {
    agent any

    environment {
        // The URL where your banking application will be hosted
        APP_URL = 'http://localhost'
    }

    stages {
        stage('Pull Latest Code') {
            steps {
                // Jenkins automatically pulls your code from your GitHub repository
                checkout scm
            }
        }

        stage('Clean Environment') {
            steps {
                echo 'Stopping any previously running application instances...'
                // Removes old running containers to free up network ports
                sh 'docker-compose down --remove-orphans'
            }
        }

        stage('Launch Martian Bank') {
            steps {
                echo 'Building and starting Martian Bank microservices via Docker...'
                // Builds the images and boots them up in detached (background) mode
                sh 'docker-compose up -d --build'
            }
        }

        stage('App Health Check') {
            steps {
                echo 'Waiting for the web application UI to become responsive...'
                // Checks if the login page returns a successful status code before finishing
                timeout(time: 3, unit: 'MINUTES') {
                    sh '''
                        until $(curl --output /dev/null --silent --head --fail ${APP_URL}); do
                            printf '.'
                            sleep 5
                        done
                        echo "Martian Bank is fully up, stable, and running!"
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment encountered an error. Gathering container logs...'
            sh 'docker-compose logs --tail=50'
        }
    }
}