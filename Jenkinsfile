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
                    def containerExists = sh(script: "docker ps -a -f name=${container_name} --format '{{.Names}}'", returnStdout: true).trim()
                    if (containerExists == container_name) {
                        echo 'Stopping and removing the container...'
                        sh '''
                            docker stop ${container_name}
                            docker rm ${container_name}
                        '''
                    } else {
                        echo 'Container ${container_name} does not exist or is not running.'
                    }
                }
                // Always attempt to remove the image if it exists
                sh '''
                    docker rmi ${image_name}:${version} || echo "Image ${image_name}:${version} does not exist."
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
               sh 'docker build -t ${image_name}:${version} ./ '
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
                    echo "pushing image cinta96/${image_name}:${version}"
                    sh "docker push cinta96/${image_name}:${version}"
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