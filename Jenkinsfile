pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'your-docker-repo/your-app:${BUILD_NUMBER}'
        DOCKER_REGISTRY_CREDENTIALS = credentials('docker-hub-credentials')
        REGISTRY_URL = "https://index.docker.io/v1/"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the source code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    echo 'Running unit and integration tests...'
                    // Example of running tests inside Docker container
                    sh "docker run --rm ${DOCKER_IMAGE} ./run_tests.sh"
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                script {
                    echo 'Logging into Docker registry...'
                    sh "docker login -u ${DOCKER_REGISTRY_CREDENTIALS_USR} -p ${DOCKER_REGISTRY_CREDENTIALS_PSW} ${REGISTRY_URL}"

                    echo 'Pushing Docker image to registry...'
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'staging'
            }
            steps {
                script {
                    echo 'Deploying to staging environment...'
                    // Assuming Kubernetes deployment here
                    sh "kubectl set image deployment/your-app-deployment your-app=${DOCKER_IMAGE} --record"
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                script {
                    echo 'Deploying to production environment...'
                    sh "kubectl set image deployment/your-app-deployment your-app=${DOCKER_IMAGE} --record"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh "docker rmi ${DOCKER_IMAGE}"
        }

        success {
            echo 'Build and deployment succeeded!'
            mail to: 'your-email@example.com',
                 subject: "SUCCESS: Build ${env.BUILD_NUMBER}",
                 body: "The build ${env.BUILD_NUMBER} completed successfully."
        }

        failure {
            echo 'Build failed!'
            mail to: 'your-email@example.com',
                 subject: "FAILURE: Build ${env.BUILD_NUMBER}",
                 body: "The build ${env.BUILD_NUMBER} failed. Please check the Jenkins logs."
        }
    }
}
