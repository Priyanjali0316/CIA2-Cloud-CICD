
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'priyanjali0316/cia2-cicd-app'
        DOCKER = 'C:\\Users\\0316a\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
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
                echo 'Building the Spring Boot application and running tests'
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
                echo 'Logging in to Docker Hub and pushing image'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    powershell '''
                        $env:DOCKER_PASSWORD |
                            & "$env:DOCKER" login `
                            --username "$env:DOCKER_USERNAME" `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            exit $LASTEXITCODE
                        }
                    '''

                    bat '"%DOCKER%" push %DOCKER_IMAGE%:%BUILD_NUMBER%'

                    bat '"%DOCKER%" push %DOCKER_IMAGE%:latest'

                    bat '"%DOCKER%" logout'
                }
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully! Docker image pushed to Docker Hub.'
        }

        failure {
            echo 'Pipeline failed. Check the Console Output for details.'
        }
    }
}
