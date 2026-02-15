pipeline {

    agent none

    environment {
        ID_DOCKER = "${ID_DOCKER_PARAMS}"
        IMAGE_NAME = "paymybuddy"
        IMAGE_TAG = "latest"
        SONAR_TOKEN = credentials('sonar_token')
        SONAR_PROJECT_KEY = "ubiakoup_mini-projet-jenkins"
        SONAR_ORG = "ubiakoup-1"
        SONAR_HOST_URL = "https://sonarcloud.io"
        SSH_USER = "ubuntu"
        HOSTNAME_DEPLOY_STAGING = "ec2-52-87-181-138.compute-1.amazonaws.com"
        HOSTNAME_DEPLOY_PROD= "ec2-54-85-137-42.compute-1.amazonaws.com"
        EC2_PUBLIC_IP_STAGING= "52.87.181.138"
        EC2_PUBLIC_IP_STAGING_PROD= "54.85.137.42"
        SPRING_DATASOURCE_USERNAME= credentials('DB_USER')
        SPRING_DATASOURCE_PASSWORD= credentials('DB_PASS')
        STAGING = "${ID_DOCKER}-staging"
        PRODUCTION = "${ID_DOCKER}-production"
    }

    stages{

        stage('Tests + Sonar') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                }
            }
            steps {
                sh '''
                mvn clean verify sonar:sonar \
                -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                -Dsonar.organization=$SONAR_ORG \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_TOKEN \
                -Dsonar.qualitygate.wait=false
                '''
            }
        }
    stage('Build & Push') {
             agent{
                 docker {
                    image 'docker:26-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            environment {
            DOCKERHUB_PASSWORD  = credentials('dockerhub')
            }
            steps {
                script {
                    sh '''
                    docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .
                    echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
                    docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
             }
        }

        stage('Deploy to Staging') {
            agent any
            steps {
                sshagent(['ssh_key']) {
                    sh '''
                    echo "Prepare remote directory"
                    ssh -o StrictHostKeyChecking=no $SSH_USER@$EC2_PUBLIC_IP_STAGING "mkdir -p /home/$SSH_USER/init-db"
        
                    echo "Copy SQL files"
                    scp -o StrictHostKeyChecking=no \
                    src/main/resources/database/*.sql \
                    $SSH_USER@$EC2_PUBLIC_IP_STAGING:/home/$SSH_USER/init-db/
        
                    echo "Deploy on EC2"
                    ssh -o StrictHostKeyChecking=no $SSH_USER@$EC2_PUBLIC_IP_STAGING <<EOF
        
                    echo "Create network"
                    docker network create paymybuddy-net || true
        
                    echo "Stop old containers"
                    docker stop paymybuddy-staging || true
                    docker rm paymybuddy-staging || true
                    docker stop mysql-staging || true
                    docker rm mysql-staging || true
        
                    echo "Create volume"
                    docker volume create mysql-staging-data || true
        
                    echo "Run MySQL"
                    docker run -d \
                      --name mysql-staging \
                      --network paymybuddy-net \
                      -e MYSQL_ROOT_PASSWORD=password \
                      -e MYSQL_DATABASE=db_paymybuddy \
                      -v mysql-staging-data:/var/lib/mysql \
                      -v /home/$SSH_USER/init-db:/docker-entrypoint-initdb.d \
                      mysql:8
        
                    echo "Waiting for MySQL..."
                    sleep 20
        
                    echo "Pull image"
                    docker pull ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
        
                    echo "Run App"
                    docker run -d \
                      --name paymybuddy-staging \
                      --network paymybuddy-net \
                      -p 8081:8080 \
                      -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql-staging:3306/db_paymybuddy \
                      -e SPRING_DATASOURCE_USERNAME=$SPRING_DATASOURCE_USERNAME \
                      -e SPRING_DATASOURCE_PASSWORD=$SPRING_DATASOURCE_PASSWORD \
                      ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                      
EOF
'''
                }
            }
        }


            
     }
  }
