pipeline {
    agent any

    parameters {
        string(name: 'container_name', defaultValue: 'web_server', description: 'Name of the Docker container')
        string(name: 'image_name', defaultValue: 'webserver', description: 'Name of the Docker image')
        string(name: 'version', defaultValue: 'v1', description: 'Version or tag of the Docker image')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('cinta96_dockerhub_token') // Use the ID from Jenkins credentials
        // IMAGE_NAME = 'cinta96/${image_name}'
        // TAG = '${version}'
    }


    stages {
        stage('List') {
            steps {
                sh 'ls'
            }
        }

        stage('Process_List') {
            steps {
                sh 'docker ps -a'
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    def containerExists = sh(script: "docker ps -a -f name=${params.container_name} --format '{{.Names}}'", returnStdout: true).trim()
                    if (containerExists == params.container_name) {
                        echo 'Stopping and removing the container...'
                        sh '''
                            docker stop ${params.container_name}
                            docker rm ${params.container_name}
                        '''
                    } else {
                        echo "Container ${params.container_name} does not exist or is not running."
                    }
                }
                // Always attempt to remove the image if it exists
                sh '''
                    docker rmi ${params.image_name}:${params.version} || echo "Image ${params.image_name}:${params.version} does not exist."
                '''
            }
        }

        stage('Process_List2') {
            steps {
               sh 'docker ps -a'
            }
        }
        
        stage('Pull_repo') {
            steps {
               sh 'git pull origin main'
            }
        }

        stage('Build_image') {
            steps {
                script {
                    def imageTag = "cinta96/${params.image_name}:${params.version}"
                    sh "docker build -t ${imageTag} ./"
                }
            }
        }

        stage('tag_image'){
            steps{
                sh 'docker tag ${image_name}:${version} cinta96/${image_name}:${version}'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to Docker Hub
                    sh "docker push ${IMAGE_NAME}:${TAG}"
                }
            }
        }

        stage('remove_image'){
            steps {
                sh 'docker rmi ${image_name}:${version}'
            }
        }

        stage('pull_image'){
            steps{
                sh 'docker pull cinta96/${image_name}:${version}'
            }
        }

        stage('Create_Container') {
            steps {
               sh 'docker run -dit --name ${container_name} -p 85:80 ${image_name}:${version}'
            }
        }

        stage('Display_IP') {
            steps {
                sh 'docker inspect ${container_name} | grep IP'
            }
        }
    }
}