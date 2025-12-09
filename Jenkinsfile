pipeline {
    agent any

    tools {
        // Must match the Maven name in "Global Tool Configuration"
        maven 'M2_HOME'
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
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=student-management \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.token=$SONAR_TOKEN
                    '''
                }
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

        stage('Deploy to Kubernetes') {
            steps {
                // Jenkins user must have ~/.kube and ~/.minikube set up (we already did that)
                sh '''
                    echo "Current kube context:"
                    kubectl config current-context || echo "No context"

                    # Create namespace devops if it doesn't exist
                    kubectl get ns devops || kubectl create namespace devops

                    echo "Apply MySQL & Spring Boot manifests"
                    kubectl apply -f mysql-deployment.yaml -n devops
                    kubectl apply -f spring-deployment.yaml -n devops

                    echo "Update deployment image to the new build"
                    kubectl set image deployment/spring-deployment \
                        spring-app=${DOCKER_IMAGE}:${BUILD_NUMBER} -n devops

                    echo "Wait for rollout to finish"
                    kubectl rollout status deployment/spring-deployment -n devops

                    echo "Pods in devops namespace:"
                    kubectl get pods -n devops

                    echo "Services in devops namespace:"
                    kubectl get svc -n devops
                '''
            }
        }
    }
}
