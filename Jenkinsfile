pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myapp"
        EC2_IP = "18.234.35.231"
        EC2_USER = "ec2-user"
        // Update this path to where your private SSH key actually is on your Windows machine
        SSH_KEY = "C:/Users/USER/.ssh/jenkins_id_rsa"
        // Path to bash.exe from Git for Windows
        BASH_PATH = "C:/Program Files/Git/usr/bin/bash.exe"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/mahadevprsd7/my-docker-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Run docker build with Git Bash shell
                // Use triple quotes for multiline, and ${} for variable interpolation in Groovy
                powershell(script: """
                    & '${BASH_PATH}' -c 'docker build -t ${DOCKER_IMAGE} .'
                """)
            }
        }

        stage('Deploy to EC2') {
            steps {
                powershell(script: """
                    & '${BASH_PATH}' -c \"
                    scp -i ${SSH_KEY} -o StrictHostKeyChecking=no Dockerfile index.html styles.css ${EC2_USER}@${EC2_IP}:/home/ec2-user/ &&
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        cd /home/ec2-user &&
                        docker build -t ${DOCKER_IMAGE} . &&
                        docker stop myapp-container || true &&
                        docker rm myapp-container || true &&
                        docker run -d -p 80:80 --name myapp-container ${DOCKER_IMAGE}
                    '
                    \"
                """)
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}
