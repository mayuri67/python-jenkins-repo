pipeline {

    agent any

    environment {

        DOCKER_IMAGE = "mayuri67/python-jenkins"
        IMAGE_TAG = "latest"

    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/mayuri67/python-jenkins-repo.git', credentialsId: 'githubcredentials'

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
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login \
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
            echo "Deploying Python application to MicroK8s"

            /snap/bin/microk8s kubectl apply -f k8s/deployment.yaml

            /snap/bin/microk8s kubectl apply -f k8s/service.yaml

            echo "Checking deployment status"

            /snap/bin/microk8s kubectl get deployments

            /snap/bin/microk8s kubectl get pods

            /snap/bin/microk8s kubectl get services

            /snap/bin/microk8s kubectl rollout restart deployment/python-app
            /snap/bin/microk8s kubectl rollout status deployment/python-app
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