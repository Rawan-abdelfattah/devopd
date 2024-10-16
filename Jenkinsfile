pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub' // Your Docker Hub credentials ID
        DOCKER_IMAGE_NAME = 'web_app' // Name for the Docker image
        ANSIBLE_PLAYBOOK = 'deploy.yml' // Ansible playbook (not used in the build stage)
        TERRAFORM_DIR = 'terraform/'  // Directory containing your Terraform configuration
    }

    stages {
        stage('Cleanup') {
            steps {
                cleanWs() // Clean workspace before starting
            }
        }

        stage('Checkout') {
            steps {
                // Check out the code from the specified Git repository
                // git branch: 'main', credentialsId: 'Git', url: 'https://github.com/Rawan-abdelfattah/devops'
                git branch: 'main', url: 'https://github.com/Rawan-abdelfattah/devops'

            }
        }

        stage('key generation'){
            steps{
                sh '''
                cd terraform
                ssh-keygen -f mykey
                cp mykey ~/.ssh/
                cp mykey.pub ~/.ssh/
                cd ..
                '''
            }
        }


        stage('Terraform Init') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    dir(TERRAFORM_DIR) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    dir(TERRAFORM_DIR) {
                        sh 'terraform plan -out=tfplan'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    dir(TERRAFORM_DIR) {
                        sh 'terraform apply -input=false tfplan'
                    }
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                // Use Docker Hub credentials to log in
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    // Log in to Docker Hub
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    
                    // Build the Docker image using Docker Compose
                    sh 'docker-compose build' // Include --verbose for more detailed output
                }
            }
        }
    }

    post {
        always {
            // Logout from Docker after the build is complete to clean up
            sh 'docker logout'
        }
    }
}
