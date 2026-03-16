pipeline {
    agent any

    environment {
        CONTAINER_NAME = "react-cart-container"
        PORT = "3000"
    }
    tools {
    nodejs "node20"
}

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'Git-Creds',
                        url: 'https://github.com/BadamTeja/reactjs-shopping-cart.git'
                    ]]
                )
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Tests') {
            steps {
                sh 'npm test -- --watchAll=false || true'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker build -t $DOCKER_USER/react-shopping-cart:${BUILD_NUMBER} .

                    docker push $DOCKER_USER/react-shopping-cart:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Remove Old Container') {
            steps {
                sh '''
                docker stop react-cart-container || true
                docker rm react-cart-container || true
                '''
            }
        }

        stage('Run New Container') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    docker run -d -p 3000:80 \
                    --name react-cart-container \
                    $DOCKER_USER/react-shopping-cart:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Cleanup Old Images') {
            steps {
                sh '''
                docker image prune -f
                '''
            }
        }
    }
}
