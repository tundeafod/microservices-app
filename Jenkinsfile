pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/currencyservice'
        GIT_PASSWORD = 'git-password'
        GIT_USERNAME = 'git-username'
        GITHUB_CREDENTIALS_ID = 'git-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    def majorVersion = '1'
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    def formattedBuildNumber = String.format('%02d', buildNumber)
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    env.NEW_DOCKER_IMAGE = "${DOCKER_IMAGE}:${imageTag}"
                    withDockerRegistry(credentialsId: DOCKER_HUB_CREDENTIALS, toolName: 'docker') {
                        sh "docker build -t ${NEW_DOCKER_IMAGE} ."
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_HUB_CREDENTIALS, toolName: 'docker') {
                        sh "docker push ${NEW_DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    // Clone the repository
                    git branch: 'main', credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/tundeafod/microservices-app.git'

                    // Use sed to update the deployment-service.yml file with the new Docker image tag
                    def majorVersion = '1'
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    def formattedBuildNumber = String.format('%02d', buildNumber)
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    env.NEW_DOCKER_IMAGE = "${DOCKER_IMAGE}:${imageTag}"
                    sh "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${env.NEW_DOCKER_IMAGE}|' deployment-service.yml"
                    

                    // Commit and push the changes
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            git add deployment-service.yml
                            git commit -m 'Update image tag to ${env.DOCKER_IMAGE}:${imageTag}'
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git main
                        """
                    }
                }
            }
        }

        stage('Clean up disk') {
            steps {
                script {
                    sh "docker rmi ${NEW_DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
