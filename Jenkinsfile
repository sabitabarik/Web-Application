pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sabitabarik/web-app:${BUILD_NUMBER}"
        GIT_REPO_NAME = "Web-Application"
        GIT_USER_NAME = "sabitabarik"
        DEPLOYMENT_FILE_PATH = "Kubernetes/deployment.yml"  // Adjust the path if necessary
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sabitabarik/Web-Application.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker_id', toolName: 'Docker') {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }
        
        stage('Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker_id', toolName: 'Docker') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        
        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github_id', variable: 'GITHUB_TOKEN')]) {
                    script {
                        sh '''
                            git config user.email "sabita.niki22@gmail.com"
                            git config user.name "Sabita Barik"
                        '''

                        sh 'git checkout main'
                        sh 'cat ${DEPLOYMENT_FILE_PATH}'

                        sh '''
                            sed -i "s|image: sabitabarik/web-app:[^[:space:]]*|image: sabitabarik/web-app:${BUILD_NUMBER}|g" ${DEPLOYMENT_FILE_PATH}
                        '''

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
        
        stage('Deploy to EKS') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_Credetilas_id']]) {
                        // Configure kubectl with EKS credentials
                        sh '''
                            aws eks --region ap-south-1 describe-cluster --name devopsshack-cluster --query cluster.status
                            aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster
                            
                            # Create necessary namespaces
                            if ! kubectl get namespace webapps; then
                                kubectl create namespace webapps
                            else
                                echo "Namespace 'webapps' already exists."
                            fi

                            # Apply other resources
                            
                            kubectl apply -f ${DEPLOYMENT_FILE_PATH} -n webapps
                            kubectl apply -f Kubernetes/service.yml -n webapps
                        '''
                        sleep 20
                    }
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_Credetilas_id']]) {
                        sh '''
                            kubectl get pods -n webapps
                            kubectl get svc -n webapps
                        '''
                    }
                }
            }
        }
    }
}
