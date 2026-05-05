pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'egxsmit/healthcare-app'
        DOCKER_TAG   = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Unit Tests') {
            steps {
                bat 'if not exist reports mkdir reports'
                bat 'set PYTHONPATH=%CD% && pytest tests/ --junitxml=reports/test-results.xml --cov=app --cov-report=xml:reports/coverage.xml'
            }
            post {
                always {
                    junit 'reports/test-results.xml'
                }
            }
        }

        // ✅ SAST (non-blocking)
        stage('SAST - Static Code Analysis') {
            steps {
                script {
                    echo 'Running Semgrep...'
                    bat '''
                        docker run --rm ^
                        -v %CD%:/src ^
                        returntocorp/semgrep semgrep ^
                        --config=p/python ^
                        /src || echo SAST completed
                    '''
                }
            }
        }

        // ✅ BUILD FIRST (IMPORTANT FIX)
        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        // ✅ TRIVY AFTER BUILD (FIXED)
        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    echo 'Scanning Docker image with Trivy...'
                    bat '''
                        docker run --rm ^
                        -v //var/run/docker.sock:/var/run/docker.sock ^
                        aquasec/trivy:latest image ^
                        --exit-code 0 ^
                        --severity HIGH,CRITICAL ^
                        --format table ^
                        egxsmit/healthcare-app:latest || echo Trivy scan completed
                    '''
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                    bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s/deployment.yaml'
                bat 'kubectl apply -f k8s/service.yaml'
                bat 'kubectl rollout status deployment/healthcare-app'
            }
        }

        stage('Verification') {
            steps {
                script {
                    echo 'Verifying deployment...'
                    bat 'kubectl get pods'
                    bat 'kubectl get svc healthcare-service'
                }
            }
        }
    }

    post {
        always  { echo 'Pipeline completed. Reports available.' }
        success { echo 'DevSecOps pipeline SUCCESS ✅' }
        failure { echo 'Pipeline FAILED ❌' }
    }
}
