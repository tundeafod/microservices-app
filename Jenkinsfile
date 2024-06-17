pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/adservice'
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
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    def majorVersion = '1'
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    def formattedBuildNumber = String.format('%02d', buildNumber)
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    def deploymentFile = 'tundeafod/microservices-app:main/deployment-service.yml'
        
                    sh """
                    sed -i 's|image: adijaiswal/emailservice:.*|image: adijaiswal/emailservice:${imageTag}|' deployment-service.yml
                    """
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
