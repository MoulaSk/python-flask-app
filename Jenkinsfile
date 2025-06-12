pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup') {
            steps {
             sh '''
                    /usr/local/bin/python3.10 -m venv venv
                    source venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    source venv/bin/activate
                    python -m pytest
                '''
            }
        }

        stage('Package Code') {
            steps {
                sh '''
                    zip -r python.zip ./* -x '*.git*'
                    ls -lart
                '''
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                        scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no python.zip ${username}@${SERVER_IP}:/home/ec2-user/
                        ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << 'EOF'
                            set -e
                            mkdir -p /home/ec2-user/app
                            unzip -o /home/ec2-user/python.zip -d /home/ec2-user/app/
                            cd /home/ec2-user/app
                            python3 -m venv venv --clear
                            source venv/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt --no-cache-dir
                            sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment completed!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs."
        }
    }
}
