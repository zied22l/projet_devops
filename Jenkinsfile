pipeline {
    agent any

    tools {
        maven 'M2_HOME'   // must match Maven name in Jenkins
    }

    environment {
        DOCKER_IMAGE = 'zied123alimi/projet-devops'
    }

    stages {

        stage('GIT') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/zied22l/projet_devops.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('MVN SONARQUBE') {
            steps {
                sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=student-management \
                      -Dsonar.host.url=http://localhost:9000 \
                      -Dsonar.login=admin \
                      -Dsonar.password=Ziedbensalah123$
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
                sh 'docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
    }
}

