pipeline {
    environment {
        DOCKER_ID = "seily" // Remplacez cela par votre ID Docker
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBECONFIG = credentials("config")
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    agent any

    stages {
        stage('Build and run movie-service') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG ./movie-service
                    sleep 6
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Build and run cast-service') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins-cast
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG ./cast-service
                    sleep 6
                    docker run -d -p 8088:8080 --name jenkins-cast $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh 'curl localhost'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deployment in dev') {
            steps {
                script {
                    sh '''
                    mkdir -p ~/.kube
                    cp $KUBECONFIG ~/.kube/config
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app movie-service --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deployment in staging') {
            steps {
                script {
                    sh '''
                    mkdir -p ~/.kube
                    cp $KUBECONFIG ~/.kube/config
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app movie-service --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deployment in qa') {
            steps {
                script {
                    sh '''
                    mkdir -p ~/.kube
                    cp $KUBECONFIG ~/.kube/config
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app movie-service --values=values.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deployment in prod') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    mkdir -p ~/.kube
                    cp $KUBECONFIG ~/.kube/config
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app movie-service --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
