@Library('fatbinetou') _
pipeline {
    environment {
        IMAGE_NAME      = 'webapp'
        IMAGE_TAG       = 'v1'
        DOCKER_PASSWORD = credentials('docker-password')
        DOCKER_USERNAME = 'fatimal23'
        HOST_PORT       = 80
        CONTAINER_PORT  = 80
        IP_DOCKER       = '172.17.0.1'
    }

    agent any

    stages {

        stage('Build') {
            steps {
                script {
                    sh '''
                        docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                        docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                        curl -I http://$IP_DOCKER:$HOST_PORT
                        sleep 5
                        docker stop $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    sh '''
                        docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Review') {
            environment {
                SERVEUR_IP = '16.16.160.63'
                SERVEUR_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: 'MINUTES') {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement *Review* ?", ok: 'Oui'
                    }
                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker rm -f $IMAGE_NAME || echo 'Container not found'"
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker run -d -p $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            curl -I http://$SERVEUR_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        stage('Deploy Staging') {
            environment {
                SERVEUR_IP = '16.16.74.192'
                SERVEUR_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: 'MINUTES') {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement *Staging* ?", ok: 'Oui'
                    }
                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker rm -f $IMAGE_NAME || echo 'Container not found'"
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker run -d -p $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            curl -I http://$SERVEUR_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        stage('Deploy Prod') {
            when {
                expression {
                    return env.BRANCH_NAME == 'origin/prod'
                }
            }
            environment {
                SERVEUR_IP = '16.170.167.67'
                SERVEUR_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: 'MINUTES') {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement *Production* ?", ok: 'Oui'
                    }
                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker rm -f $IMAGE_NAME || echo 'Container not found'"
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            ssh -o StrictHostKeyChecking=no $SERVEUR_USERNAME@$SERVEUR_IP "docker run -d -p $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            curl -I http://$SERVEUR_IP:$HOST_PORT
                        '''
                    }
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
