pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        REPO_NAME = 'afod2000'
        DOCKER_IMAGES = 'adservice'
        DEPLOYMENT_YAML_PATH = './main/deployment-service.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Set Permissions') {
            steps {
                sh 'chmod 644 ${DEPLOYMENT_YAML_PATH}'
            }
        }

        stage('Build, Push, Update YAML for Each Image') {
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
                            sh "docker build -t ${fullImageName} ."

                            // Push Docker Image
                            sh "docker push ${fullImageName}"

                            // Remove Docker Image from local storage
                            sh "docker rmi ${fullImageName}"

                            // Update deployment-service.yml
                            sh """
                                sed -i 's#${REPO_NAME}/${imageName}:.*#${fullImageName}#' ${DEPLOYMENT_YAML_PATH}
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
