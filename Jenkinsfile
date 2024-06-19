pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/productcatalogservice'
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
        
        stage('Update deployment-service.yml') {
            steps {
                script {
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    def dockerImageName = "${DOCKER_IMAGE}:${imageTag}"
                    
                    // Read deployment-service.yml
                    def yamlFile = readFile('deployment-service.yml')
                    
                    // Update the image line with the new Docker image name
                    yamlFile = yamlFile.replaceAll(/image: afod2000\/productcatalogservice:\d+\.\d+/, "image: ${dockerImageName}")
                    
                    // Write the updated yaml back to file
                    writeFile(file: 'deployment-service.yml', text: yamlFile)
                    
                    // Stage the updated deployment-service.yml for commit
                    sh "git add deployment-service.yml"
                    
                    // Commit the changes
                    sh "git commit -m 'Update deployment-service.yml with new Docker image ${dockerImageName}'"
                    
                    // Push the changes back to the repository
                    sh "git push origin main"
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
