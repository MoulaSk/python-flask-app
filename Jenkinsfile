pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        stage('Setup') {
            steps {
            sh "pip install -r requirements.txt"

            }
        }
        stage('Test') {
            steps {
             sh "pytest"
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"

            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'MY_SSH_KEY', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                        scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/

                        ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << 'EOF'
                            cd /home/ec2-user/
                            rm -rf app/
                            unzip -o myapp.zip -d app/

                            cd app/
                            python3 -m venv venv
                            source venv/bin/activate
                            pip install -r requirements.txt

                            sudo systemctl restart flaskapp.service || echo "Service not found"
                        EOF
                    '''
                }
            }
        }
    }
}
