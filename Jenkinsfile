pipeline {

    agent none

    environment {
        ID_DOCKER = "${ID_DOCKER_PARAMS}"
        IMAGE_NAME = "paymybuddy"
        IMAGE_TAG = "latest"
        SONAR_TOKEN = credentials('sonar_token')
        SONAR_PROJECT_KEY = "paymybuddy_project"
        SONAR_ORG = "ubiakoup"
        SONAR_HOST_URL = "https://sonarcloud.io"

        STAGING = "${ID_DOCKER}-staging"
        PRODUCTION = "${ID_DOCKER}-production"
    }

    stages{

        stage('Tests + Sonar') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                    args '-v /var/run/docker.sock:/var/run/docker.sock --network host'
                }
            }
            steps {
                sh '''
                docker run -d --name mysql-test \
                -e MYSQL_ROOT_PASSWORD=password \
                -e MYSQL_DATABASE=db_paymybuddy \
                -v $PWD/src/main/resources/database:/docker-entrypoint-initdb.d \
                -p 3306:3306 mysql:8

                echo "Waiting for MySQL..."
                sleep 10

                ./mvnw clean verify sonar:sonar \
                -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                -Dsonar.organization=$SONAR_ORG \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
    }

}
