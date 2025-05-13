@Libary('fatbinetou')
pipeline {
    // Déclaration des variables d'environnement accessibles dans toutes les étapes
    environment {
        IMAGE_NAME      = 'webapp'                      // Nom de l'image Docker
        IMAGE_TAG       = 'v1'                          // Tag de version de l'image
        DOCKER_PASSWORD = credentials('docker-password') // Récupération du mot de passe DockerHub stocké dans Jenkins
        DOCKER_USERNAME = 'fatimal23'                   // Nom d'utilisateur DockerHub
        HOST_PORT       = 80                         // Port utilisé sur la machine hôte
        CONTAINER_PORT  = 80                            // Port exposé par le container
        IP_DOCKER       = '172.17.0.1'                  // Adresse IP locale de Docker
    }

    agent any // Exécuter le pipeline sur n'importe quel agent disponible

    stages {

        // Étape de build de l'image Docker
        stage('Build') {
            steps {
                script {
                    sh '''
                        docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        // Étape de test de l'image Docker localement
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

        // Étape de publication de l'image sur DockerHub
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

        // Déploiement sur l'environnement de review
        stage('Deploy Review') {
            environment {
                SERVEUR_IP = '16.16.160.63'       // IP du serveur Review
                SERVEUR_USERNAME = 'ubuntu'   // Nom d'utilisateur SSH
            }
            steps {
                script {
                     // Timeout pour attendre la confirmation manuelle du déploiement
                            timeout(time: 30, unit: 'MINUTES') {
                                input message: "Voulez-vous réaliser un déploiement sur l'environnement de production ?", ok: 'Yes'
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

        // Déploiement sur l'environnement de staging
        stage('Deploy Staging') {
            environment {
                SERVEUR_IP = '16.16.74.192'       // IP du serveur Staging
                SERVEUR_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    // Timeout pour attendre la confirmation manuelle du déploiement
                        timeout(time: 30, unit: 'MINUTES') {
                            input message: "Voulez-vous réaliser un déploiement sur l'environnement de production ?", ok: 'Yes'
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

        // Déploiement sur l'environnement de production
        stage('Deploy Prod') {
            when {expression {GIT_BRANCH == 'oring/prod'}}
            environment {
                SERVEUR_IP = '16.170.167.67'       // IP du serveur de production
                SERVEUR_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    // Timeout pour attendre la confirmation manuelle du déploiement
                    timeout(time: 30, unit: 'MINUTES') {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement de production ?", ok: 'Yes'
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

    // Bloc exécuté en fin de pipeline, peu importe le résultat
    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        }
    }
}
