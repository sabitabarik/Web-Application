pipeline {
    agent any


    environment {
        DOCKERHUB_CREDENTIALS = credentials('Docker_id')
        DOCKER_IMAGE = "manoj3003/pipelie-cicd:${BUILD_NUMBER}"
        OLD_IMAGE_TAG_PATTERN = "manoj3003/pipelie-cicd:*"  // Pattern to match old images
        GIT_REPO_NAME = "swiggy-nodejs-devops-project"
        GIT_USER_NAME = "manoj7894"
        DEPLOYMENT_FILE_PATH = "Kubernetes/deployment.yml"  // Adjust the path if necessary
    }

    stages {
        stage('GetCode') {
            steps {
                git branch: 'main', url: 'https://github.com/manoj7894/swiggy-nodejs-devops-project.git'
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    def dockerImage = docker.image(DOCKER_IMAGE)
                    dockerImage.push() // Pushes the Docker image to the repository
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github_id', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Configure Git
                        sh '''
                            git config user.email "manojvarmapotthuri3003@gmail.com"
                            git config user.name "Manojvarma Potthuri"
                        '''

                        // Check out the repository
                        sh 'git checkout main'

                        // Display the current content of the deployment file for debugging
                        sh 'cat ${DEPLOYMENT_FILE_PATH}'

                        // Update the image tag in deployment.yml
                        sh '''
                            # Use sed to replace the image tag dynamically
                            sed -i "s|image: manoj3003/pipelie-cicd:[^[:space:]]*|image: manoj3003/pipelie-cicd:${BUILD_NUMBER}|g" ${DEPLOYMENT_FILE_PATH}
                        '''

                        // Check for changes and commit if necessary
                        sh '''
                            if git diff --quiet; then
                                echo "No changes detected in ${DEPLOYMENT_FILE_PATH}"
                            else
                                git add ${DEPLOYMENT_FILE_PATH}
                                git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                                git push -f https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            emailext (
                subject: "Pipeline Status: ${BUILD_NUMBER}",
                body: '''<html>
                           <body>
                              <p>Build Status: ${BUILD_STATUS}</p>
                              <p>Build Number: ${BUILD_NUMBER}</p>
                              <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                           </body>
                        </html>''',
                to: 'varmapotthuri4@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'
            )
        }
    }
}
