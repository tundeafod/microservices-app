pipeline { 
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-creds'
        DOCKER_IMAGE = 'afod2000/adservice'
        TARGET_BRANCH = 'main'
        PROD_BRANCH = 'production'
        REPO_URL = 'https://github.com/tundeafod/microservices-app.git'
        MANIFEST_FILE_PATH = 'deployment-service.yml'
        COMMIT_MESSAGE = 'Update manifest file'
        CREDENTIALS_ID = 'git-creds'
        GIT_USERNAME = 'git-username'
        GIT_PASSWORD = 'git-password'
    }

    stages {
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

        stage('Push Docker Image') {
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
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${TARGET_BRANCH}"]],
                        userRemoteConfigs: [[url: REPO_URL, credentialsId: CREDENTIALS_ID]]
                    ])
                }
            }
        }

        stage('Update Manifest File') {
            steps {
                script {
                    def majorVersion = '1'
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    def formattedBuildNumber = String.format('%02d', buildNumber)
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    def sedCommand = "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${imageTag}|' ${MANIFEST_FILE_PATH}"

                    sh "echo ${sedCommand}"
                    sh sedCommand
                    sh "git status"
                    sh 'git config user.name "jenkins"'
                    sh 'git config user.email "jenkins@example.com"'
                    sh "git add ${MANIFEST_FILE_PATH}"
                    sh "git commit -m '${COMMIT_MESSAGE} to ${DOCKER_IMAGE}:${imageTag}'"

                    // Push the changes
                    withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                        echo "Pushing changes to GitHub"
                        git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git
                        git pull origin main
                        '''
                    }
                }
            }
        }

        stage('Checkout Production Branch') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${PROD_BRANCH}"]],
                        userRemoteConfigs: [[url: REPO_URL, credentialsId: CREDENTIALS_ID]]
                    ])
                }
            }
        }

        stage('Update Manifest Prod File') {
            steps {
                script {
                    def majorVersion = '1'
                    def buildNumber = env.BUILD_NUMBER.toInteger()
                    def formattedBuildNumber = String.format('%02d', buildNumber)
                    def imageTag = "${majorVersion}.${formattedBuildNumber}"
                    def sedCommand = "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${imageTag}|' ${MANIFEST_FILE_PATH}"

                    sh "echo ${sedCommand}"
                    sh sedCommand
                    sh "git status"
                    sh 'git config user.name "jenkins"'
                    sh 'git config user.email "jenkins@example.com"'
                    sh "git add ${MANIFEST_FILE_PATH}"
                    sh "git commit -m '${COMMIT_MESSAGE} to ${DOCKER_IMAGE}:${imageTag}'"

                    withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tundeafod/microservices-app.git HEAD:${PROD_BRANCH}"
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
