pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/checkoutservice'
        GIT_PASSWORD = credentials('git-creds')
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
                        sh "docker build -t ${env.NEW_DOCKER_IMAGE} ."
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_HUB_CREDENTIALS, toolName: 'docker') {
                        sh "docker push ${env.NEW_DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Clean up disk') {
            steps {
                script {
                    sh "docker rmi ${env.NEW_DOCKER_IMAGE}"
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    // Clone the main branch of your repository
                    checkout([$class: 'GitSCM', branches: [[name: 'main']], userRemoteConfigs: [[url: 'https://github.com/tundeafod/microservices-app.git', credentialsId: GITHUB_CREDENTIALS_ID]]])

                    // Print the variables for debugging
                    echo "DOCKER_IMAGE: ${DOCKER_IMAGE}"
                    echo "NEW_DOCKER_IMAGE: ${env.NEW_DOCKER_IMAGE}"

                    // Print the file contents before updating
                    sh """
                        echo "Original deployment-service.yml:"
                        cat deployment-service.yml
                    """

                    // Use sed to update the deployment-service.yml file
                    sh """
                        echo "Updating deployment-service.yml with new image: ${env.NEW_DOCKER_IMAGE}"
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${env.NEW_DOCKER_IMAGE}|' deployment-service.yml || echo "sed command failed"
                        echo "Updated deployment-service.yml:"
                        cat deployment-service.yml
                    """

                    // Commit and push the changes
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            git add deployment-service.yml
                            git status
                            git commit -m "Updated deployment with new Docker image: ${env.NEW_DOCKER_IMAGE}" || echo "Nothing to commit"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git HEAD:main
                        """
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
