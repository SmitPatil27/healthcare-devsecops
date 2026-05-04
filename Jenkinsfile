pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'egxsmit/healthcare-app'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/SmitPatil27/healthcare-devsecops.git'
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
                bat 'pytest tests/ --junitxml=reports/test-results.xml --cov=app --cov-report=xml:reports/coverage.xml'
            }
            post {
                always {
                    junit 'reports/test-results.xml'
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan . --format HTML --out reports/', odcInstallation: 'OWASP-DC'
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: true,
                        reportDir: 'reports',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency Check'
                    ])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                    bat "docker push %DOCKER_IMAGE%:%BUILD_NUMBER%"
                }
            }
        }

    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure  { echo 'Pipeline failed. Check stage logs.' }
    }
}
