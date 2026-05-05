pipeline {
    agent any

    environment {
        IMAGE_NAME = "egxsmit/healthcare-app"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        KUBECONFIG = "C:\\Users\\Smit Patil\\.kube\\config"
    }

    stages {

        stage('1. Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/SmitPatil27/healthcare-devsecops.git'
                echo "Code checked out"
            }
        }

        stage('2. Install and Test') {
            steps {
                bat 'pip install -r requirements.txt'
                bat 'pytest test_app.py -v'
                echo "Tests passed"
            }
        }

        stage('3. Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                echo "Image built"
            }
        }

        stage('4. Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    bat "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                    bat "docker push ${IMAGE_NAME}:latest"
                }
                echo "Image pushed"
            }
        }

        stage('5. Deploy to Kubernetes') {
    steps {
        bat "kubectl config use-context docker-desktop"
        bat "powershell -Command \"(Get-Content k8s/deployment.yaml) -replace 'BUILD_TAG','${IMAGE_TAG}' | Set-Content k8s/deployment.yaml\""
        bat "kubectl apply -f k8s/deployment.yaml --validate=false"
        bat "kubectl apply -f k8s/service.yaml --validate=false"
        bat "kubectl rollout status deployment/python-app --timeout=120s"
        echo "Deployed to Kubernetes"
    }
}

        stage('6. Verify') {
            steps {
                bat 'kubectl get pods'
                bat 'kubectl get svc'
                echo "Pipeline complete! App running at http://localhost:30080"
            }
        }
    }

    post {
        success {
            echo "PASSED - Build ${IMAGE_TAG} deployed!"
        }
        failure {
            echo "FAILED - Check logs above"
        }
        always {
            bat "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || exit 0"
        }
    }
}
