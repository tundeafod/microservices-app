pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/adservice'
        GITHUB_CREDENTIALS_ID = 'git-creds'
        GIT_TOKEN = 'git-token'
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

                    sh "docker build -t ${env.NEW_DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
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
                    withCredentials([string(credentialsId: GIT_TOKEN, variable: 'GITHUB_TOKEN')]) {
                        // Clone the main branch of your repository
                        sh "git clone --branch main https://${GITHUB_TOKEN}@github.com/tundeafod/microservices-app.git"

                        // Update the image in deployment-service.yml
                        def deploymentFile = readFile 'microservices-app/deployment-service.yml'
                        def updatedDeploymentFile = deploymentFile.replaceAll(/image: .*/, "image: ${env.NEW_DOCKER_IMAGE}")
                        writeFile file: 'microservices-app/deployment-service.yml', text: updatedDeploymentFile

                        // Commit and push the changes
                        sh '''
                        cd microservices-app
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add deployment-service.yml
                        git commit -m "Updated deployment with new Docker image: ${env.NEW_DOCKER_IMAGE}"
                        git push origin main
                        '''
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
