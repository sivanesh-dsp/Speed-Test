pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'custom_nginx_image'
        CONTAINER_NAME = 'nginx_html'
        SONARQUBE_SERVER = 'SonarQube' // Replace with your SonarQube server name in Jenkins
        PROJECT_KEY = 'poc' // Replace with your project key configured in SonarQube
        SONAR_SCANNER_HOME = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/sivanesh-dsp/Speed-Test.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=${PROJECT_KEY} -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the repository
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Stop and remove any existing container with the same name
                    sh "docker rm -f $CONTAINER_NAME || true"

                    // Run the container with the built image
                    sh "docker run -d -p 80:80 --name $CONTAINER_NAME $DOCKER_IMAGE"
                }
            }
        }

        stage('Post-Deployment Test') {
            steps {
                script {
                    // Test if the Nginx container is running and serving the content
                    sh 'curl -I http://localhost | grep "200 OK"'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
