pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/adservice'
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

                    // Read and update the deployment-service.yml file
                    def deploymentFile = readFile 'deployment-service.yml'
                    def deploymentFileLines = deploymentFile.readLines()
                    def newDeploymentFileLines = []
                    def inDeployment = false
                    def deploymentName = 'adservice'

                    deploymentFileLines.each { line ->
                        if (line.trim().startsWith('metadata:')) {
                            inDeployment = false
                        }
                        if (line.trim() == "name: ${deploymentName}") {
                            inDeployment = true
                        }
                        if (inDeployment && line.trim().startsWith('image:')) {
                            line = "        image: ${env.NEW_DOCKER_IMAGE}"
                        }
                        newDeploymentFileLines << line
                    }

                    def updatedDeploymentFile = newDeploymentFileLines.join('\n')
                    writeFile file: 'deployment-service.yml', text: updatedDeploymentFile

                    // Commit and push the changes
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            git add deployment-service.yml
                            git commit -m "Updated deployment with new Docker image: ${NEW_DOCKER_IMAGE}"
                            git pull https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git main
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git main
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
