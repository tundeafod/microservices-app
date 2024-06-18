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

        stage('Update Kubernetes Deployment') {
            steps {
                script {
                    withCredentials([string(credentialsId: GITHUB_TOKEN_ID, variable: 'GITHUB_TOKEN')]) {
                        sh '''
                        // Clone the main branch of your repository using token
                        git clone -b main https://${GITHUB_TOKEN}@github.com/tundeafod/microservices-app.git

                        # Update the image in deployment-service.yml
                        cd microservices-app
                        sed -i 's|image: .*|image: '${env.NEW_DOCKER_IMAGE}'|' deployment-service.yml

                        // Commit and push the changes
                       # Commit and push the changes
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
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
