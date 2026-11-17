pipeline {

    agent any

    environment {

        DOCKER_IMAGE = "mayuri67/python-jenkins"
        IMAGE_TAG = "${BUILD_NUMBER}"

    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/mayuri67/python-jenkins-project.git'

            }
        }

        stage('Install Dependencies') {

            steps {

                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''

            }
        }

        stage('Unit Testing') {

            steps {

                sh '''
                    . venv/bin/activate
                    pytest -v
                '''

            }
        }

        stage('Docker Build') {

            steps {

                sh '''
                    docker build -t $DOCKER_IMAGE:$IMAGE_TAG .
                '''

            }
        }

        stage('Trivy Security Scan') {

            steps {

                sh '''
                    trivy image --exit-code 1 \
                    --severity CRITICAL \
                    $DOCKER_IMAGE:$IMAGE_TAG
                '''

            }
        }

        stage('Docker Hub Push') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push $DOCKER_IMAGE:$IMAGE_TAG

                        docker logout
                    '''

                }
            }
        }

        stage('Deploy to Kubernetes') {

            steps {

                sh '''
                    kubectl apply -f k8s/deployment.yaml

                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/python-app \
                    python-app=$DOCKER_IMAGE:$IMAGE_TAG

                    kubectl rollout status deployment/python-app \
                    --timeout=180s
                '''

            }
        }

    }

    post {

        success {

            echo 'Python CI/CD Pipeline completed successfully!'

        }

        failure {

            echo 'Python CI/CD Pipeline failed!'

        }

    }

}