// Jenkinsfile using Declarative Pipeline syntax
pipeline {
    // Agent specifies where the entire pipeline will run (e.g., a specific Docker image or node)
    agent any

    // Parameters allow the user to select the deployment environment
    parameters {
        choice(name: 'TARGET_ENV', choices: ['UAT', 'PRODUCTION'], description: 'Select the environment to deploy to')
    }

    // Global and Environmental Variables setup
    environment {
        // Global Variables (set up as Jenkins Credentials in the real pipeline)
        // These are placeholders for actual secrets managed by Jenkins
        DOCKER_HUB_CREDENTIAL_ID = 'docker-hub-credentials' // ID of your Docker Hub secret in Jenkins
        AWS_CREDENTIAL_ID = 'aws-credentials'               // ID of your AWS secret in Jenkins

        // Environmental Variables for EC2 and Docker details [cite: 23, 24]
        DOCKER_IMAGE_NAME = "vishal933/dotnet-hello-world" // Replace 'vishal933' with your Docker Hub username
        DOCKER_IMAGE_TAG = "build-${BUILD_NUMBER}" // Uses Jenkins' built-in variable
        UAT_IP = '1.2.3.4' // Placeholder: Replace with actual UAT EC2 IP
        UAT_SSH_CREDENTIAL_ID = 'uat-ssh-key' // ID of UAT SSH secret in Jenkins
        PROD_IP = '5.6.7.8' // Placeholder: Replace with actual Production EC2 IP
        PROD_SSH_CREDENTIAL_ID = 'prod-ssh-key' // ID of Production SSH secret in Jenkins
    }

    // Stages define the main steps of the pipeline [cite: 14]
    stages {
        // Stage 1: Pull Source Code [cite: 15]
        stage('Checkout Code') {
            steps {
                // The 'checkout scm' step is implicit when using a Pipeline linked to a Git repository
                script {
                    echo "Checking out code from ${env.GIT_URL}"
                }
            }
        }

        // Stage 2: Build Docker Image [cite: 16]
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    // Use the 'docker' command or 'sh' to run a Docker command
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        // Stage 3: Push Image to Docker Hub [cite: 17, 26]
        stage('Push to Docker Hub') {
            steps {
                // Use Jenkins' 'withCredentials' to securely access Docker Hub credentials 
                withCredentials([usernamePassword(credentialsId: env.DOCKER_HUB_CREDENTIAL_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh "docker logout"
                }
            }
        }

        // Stage 4: Deploy to EC2 
        stage('Deploy to EC2') {
            steps {
                script {
                    def targetIP
                    def sshCredentialId

                    // Select variables based on the user's chosen parameter [cite: 37]
                    if (params.TARGET_ENV == 'UAT') {
                        targetIP = env.UAT_IP
                        sshCredentialId = env.UAT_SSH_CREDENTIAL_ID
                    } else if (params.TARGET_ENV == 'PRODUCTION') {
                        targetIP = env.PROD_IP
                        sshCredentialId = env.PROD_SSH_CREDENTIAL_ID
                    }

                    echo "Deploying to ${params.TARGET_ENV} at IP: ${targetIP}"

                    // Use Jenkins' 'sshagent' or 'sshPublisher' plugin for deployment
                    // The deployment typically involves SSHing into the EC2 instance and running Docker commands (e.g., docker pull and docker run)
                    withCredentials([sshUserPrivateKey(credentialsId: sshCredentialId, keyFileVariable: 'SSH_KEY_FILE', passphraseVariable: 'SSH_PASSPHRASE', usernameVariable: 'SSH_USER')]) {
                        // Example commands to run on the EC2 instance via SSH:
                        // 1. Log in to Docker Hub on the EC2 instance (if private image)
                        // 2. Pull the latest image: docker pull ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        // 3. Stop and remove the old container
                        // 4. Run the new container: docker run -d --name dotnet-app -p 80:8080 ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        echo "--- SSH deployment logic goes here (requires 'sshagent' or similar Jenkins plugin) ---"
                        echo "Target Command: ssh -i ${SSH_KEY_FILE} ${SSH_USER}@${targetIP} 'docker pull ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} && docker stop dotnet-app && docker rm dotnet-app && docker run -d --name dotnet-app -p 80:8080 ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}'"
                    }
                }
            }
        }

        // Stage 5: Testing & Verification [cite: 29, 30]
        stage('Verification') {
            steps {
                script {
                    def targetIP = params.TARGET_ENV == 'UAT' ? env.UAT_IP : env.PROD_IP
                    // Basic Health Check (e.g., using a curl command)
                    echo "Running basic health check on ${targetIP}"
                    // The 'sh' command below should be run AFTER the application is running on the EC2 instance
                    sh "curl --fail http://${targetIP}/api/hello"
                    echo "Deployment verification successful!"
                }
            }
        }
    }
}
