pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        REPO_NAME = 'afod2000'
        DOCKER_IMAGES = 'checkoutservice'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build, Push, Remove Image') {
            steps {
                script {
                    DOCKER_IMAGES.each { imageName ->
                        withDockerRegistry(credentialsId: DOCKER_HUB_CREDENTIALS, toolName: 'docker') {
                            def majorVersion = '1'
                            def buildNumber = env.BUILD_NUMBER.toInteger()
                            def formattedBuildNumber = String.format('%02d', buildNumber)
                            def imageTag = "${majorVersion}.${formattedBuildNumber}"
                            def fullImageName = "${REPO_NAME}/${imageName}:${imageTag}"

                            // Build Docker Image
                            def dockerBuildCmd = "docker build -t ${fullImageName} ."
                            def dockerBuild = sh(script: dockerBuildCmd, returnStatus: true)

                            if (dockerBuild != 0) {
                                error "Docker build failed for ${fullImageName}"
                            }

                            // Push Docker Image
                            sh "docker push ${fullImageName}"

                            // Check if Docker Image exists locally before removing
                            def dockerImageExists = sh(script: "docker images -q ${fullImageName}", returnStdout: true).trim()
                            if (!dockerImageExists.empty) {
                                // Remove Docker Image from local storage
                                sh "docker rmi ${fullImageName}"
                            }
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
