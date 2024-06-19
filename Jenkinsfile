pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/currencyservice'
        GIT_PASSWORD = credentials('git-password') // Use Jenkins credentials binding
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
                    env.IMAGE_TAG = "${majorVersion}.${formattedBuildNumber}"
                    env.NEW_DOCKER_IMAGE = "${DOCKER_IMAGE}:${IMAGE_TAG}"
                    
                    // Build and tag Docker image
                    withDockerRegistry(credentialsId: DOCKER_HUB_CREDENTIALS, toolName: 'docker') {
                        sh "docker build -t ${NEW_DOCKER_IMAGE} ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to Docker registry
                    withDockerRegistry(credentialsId: DOCKER_HUB_CREDENTIALS, toolName: 'docker') {
                        sh "docker push ${NEW_DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    // Clone the main branch of your repository
                    git branch: 'main', credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/tundeafod/microservices-app.git'

                    // Extract current image name from deployment-service.yml
                    def currentImage = sh(script: "grep -Po '(?<=image: )${DOCKER_IMAGE}:\\S+' deployment-service.yml | head -n 1", returnStdout: true).trim()
                    echo "Current image: ${currentImage}"

                    // Replace image name in deployment-service.yml
                    sh """
                        sed -i 's|${DOCKER_IMAGE}:\\S*|${NEW_DOCKER_IMAGE}|g' deployment-service.yml
                    """

                    // Commit and push the changes if the image name was updated
                    if (sh(returnStatus: true, script: "git diff --exit-code deployment-service.yml").status != 0) {
                        withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            sh """
                                git config user.email "jenkins@example.com"
                                git config user.name "Jenkins"
                                git add deployment-service.yml
                                git commit -m "Updated deployment with new Docker image: ${NEW_DOCKER_IMAGE}"
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git main
                            """
                        }
                    } else {
                        echo "No changes detected in deployment-service.yml. Skipping commit."
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
