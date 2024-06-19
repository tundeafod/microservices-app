pipeline { 
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'henrykingiv/adservice'
        TARGET_BRANCH = 'main'
        REPO_URL = 'https://github.com/henrykingiv/microservices-app.git'
        MANIFEST_FILE_PATH = '/home/deployment-service.yaml'
        COMMIT_MESSAGE = 'Update manifest file'
        CREDENTIALS_ID = 'git-creds'
        GIT_USERNAME = 'git-username'
        GIT_PASSWORD = 'git-password'
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

        stage('Checkout Target Branch') {
            steps {
                script {
                    // Checkout the target branch
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${env.TARGET_BRANCH}"]],
                        userRemoteConfigs: [[url: env.REPO_URL, credentialsId: env.CREDENTIALS_ID]]
                    ])
                }
            }
        }

        stage('Update Manifest File') {
            steps {
                script {
                    def manifestFile = "deployment-service.yaml"
                    def majorVersion = '1'
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    def formattedBuildNumber = String.format('%02d', buildNumber)
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    def sedCommand = "sed -i 's|image: \\${DOCKER_IMAGE}:.*|image: \\${DOCKER_IMAGE}:${imageTag}|' ${manifestFile}"
                    
                    // Print the sed command for debugging
                    sh "echo ${sedCommand}"
                    
                    // Execute the sed command
                    sh sedCommand
                    
                    // Check if the file was modified
                    sh "git status"
                    
                    // Configure git user
                    sh 'git config user.name "jenkins"'
                    sh 'git config user.email "jenkins@example.com"'
        
                    // Commit the changes
                    sh "git add ${manifestFile}"
                    sh "git commit -m 'Update image tag to ${env.DOCKER_IMAGE}:${imageTag}'"
        
                    // Push the changes
                    withCredentials([usernamePassword(credentialsId: 'git-creds', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/henrykingiv/microservices-app.git HEAD:main"
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
