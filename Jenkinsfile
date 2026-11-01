
pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Users\\0316a\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Users\\0316a\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\0316a\\.kube\\config'
        DOCKER_IMAGE = 'priyanjali0316/cia2-cicd-app'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                echo 'Building the application and running tests'
                bat 'mvn -B clean package'
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving the generated JAR file'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                bat '"%DOCKER%" build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Logging in to Docker Hub and pushing the image'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        & "$env:DOCKER" login `
                            --username "$env:DOCKER_USERNAME" `
                            --password "$env:DOCKER_PASSWORD"

                        if ($LASTEXITCODE -ne 0) {
                            exit $LASTEXITCODE
                        }
                    '''

                    bat '"%DOCKER%" push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                    bat '"%DOCKER%" push %DOCKER_IMAGE%:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying the current build to Minikube'

                bat '''
                    "%KUBECTL%" --kubeconfig "%KUBECONFIG%" apply -f k8s\\deployment.yaml
                    "%KUBECTL%" --kubeconfig "%KUBECONFIG%" apply -f k8s\\service.yaml
                    "%KUBECTL%" --kubeconfig "%KUBECONFIG%" set image deployment/cia2-cicd-app cia2-cicd-app=%DOCKER_IMAGE%:%BUILD_NUMBER%
                    "%KUBECTL%" --kubeconfig "%KUBECONFIG%" rollout status deployment/cia2-cicd-app --timeout=180s
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD pipeline failed. Check the stage logs for details.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
