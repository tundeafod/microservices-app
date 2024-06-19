pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/checkoutservice'
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

        stage('Clean up disk') {
            steps {
                script {
                    sh "docker rmi ${NEW_DOCKER_IMAGE}"
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    // Clone the main branch of your repository
                    git branch: 'main', credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/tundeafod/microservices-app.git'

                    // Print the variables for debugging
                    echo "DOCKER_IMAGE: ${DOCKER_IMAGE}"
                    echo "NEW_DOCKER_IMAGE: ${NEW_DOCKER_IMAGE}"

                    // Print the file contents before updating
                    sh """
                        echo "Original deployment-service.yml:"
                        cat deployment-service.yml
                    """

                    // Use sed to update the deployment-service.yml file
                    def sedCommand = "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${NEW_DOCKER_IMAGE}|' deployment-service.yml"
                    def sedResult = sh(script: sedCommand, returnStatus: true) // Capture the return status

                    if (sedResult != 0) {
                        error "Failed to update deployment-service.yml with sed command"
                    }

                    // Verify if the file was actually updated
                    sh """
                        echo "Updated deployment-service.yml:"
                        cat deployment-service.yml
                    """

                    // Commit and push the changes
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        def gitStatus = sh(script: 'git status', returnStatus: true) // Capture the return status of git status

                        if (gitStatus != 0) {
                            error "Failed to run git status"
                        }

                        if (gitStatus == 0) {
                            sh """
                                git config user.email "jenkins@example.com"
                                git config user.name "Jenkins"
                                git add deployment-service.yml
                                git status
                                git commit -m "Updated deployment with new Docker image: ${NEW_DOCKER_IMAGE}" || echo "Nothing to commit"
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git main
                            """
                        }
                    }
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
