pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/adservice'
        GITHUB_CREDENTIALS_ID = 'git-creds'
        GITHUB_TOKEN = 'git-token'
        GITHUB_TOKEN_ID = 'your-github-token-id'
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
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        def majorVersion = '1'
                        def buildNumber = env.BUILD_NUMBER.toInteger()
                        def formattedBuildNumber = String.format('%02d', buildNumber)
                        def imageTag = "${majorVersion}.${formattedBuildNumber}"
                        sh "docker build -t ${DOCKER_IMAGE}:${imageTag} ."
                        env.NEW_DOCKER_IMAGE = "${DOCKER_IMAGE}:${imageTag}"
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        def majorVersion = '1'
                        def buildNumber = env.BUILD_NUMBER.toInteger()
                        def formattedBuildNumber = String.format('%02d', buildNumber)
                        def imageTag = "${majorVersion}.${formattedBuildNumber}"
                        sh "docker push ${DOCKER_IMAGE}:${imageTag}"
                    }
                }
            }
        }

        stage('Clean up disk') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        def majorVersion = '1'
                        def buildNumber = env.BUILD_NUMBER.toInteger()
                        def formattedBuildNumber = String.format('%02d', buildNumber)
                        def imageTag = "${majorVersion}.${formattedBuildNumber}"
                        sh "docker rmi ${DOCKER_IMAGE}:${imageTag}"
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    withCredentials([string(credentialsId: GITHUB_TOKEN_ID, variable: 'GITHUB_TOKEN')]) {
                        // Clone the main branch of your repository using token
                        sh "git clone https://${GITHUB_TOKEN}@github.com/tundeafod/microservices-app.git -b main"

                        // Update the image in deployment-service.yml
                        def deploymentFile = readFile 'deployment-service.yml'
                        def updatedDeploymentFile = deploymentFile.replaceAll(/image: .*/, "image: ${env.NEW_DOCKER_IMAGE}")
                        writeFile file: 'deployment-service.yml', text: updatedDeploymentFile

                        // Commit and push the changes
                        sh '''
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add deployment-service.yml
                        git commit -m "Updated deployment with new Docker image: ${NEW_DOCKER_IMAGE}"
                        git push https://${GITHUB_TOKEN}@github.com/tundeafod/microservices-app.git main
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
