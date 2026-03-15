pipeline {
    agent any


    environment {
        DOCKER_IMAGE = "yourdockerhub/react-shopping-cart"
        CONTAINER_NAME = "react-cart-container"
        PORT = "3000"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'Git-Creds', url: 'https://github.com/BadamTeja/reactjs-shopping-cart.git']])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false || true'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            }
        }

        stage('Remove Old Container') {
            steps {
                sh '''
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh """
                docker run -d -p ${PORT}:80 \
                --name ${CONTAINER_NAME} \
                ${DOCKER_IMAGE}:${BUILD_NUMBER}
                """
            }
        }

        stage('Cleanup Old Images') {
            steps {
                sh """
                docker images ${DOCKER_IMAGE} --format "{{.Repository}}:{{.Tag}}" | grep -v ${BUILD_NUMBER} | xargs -r docker rmi || true
                """
            }
        }

    }
}
