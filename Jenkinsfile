pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        stage ("Clean WS") {
            steps {
                deleteDir()
            }
        }
        stage("checkout") {
            steps {
                checkout scm
            }
        }
        stage('Setup') {
            steps {
            sh "pip install -r requirements.txt"

            }
        }
        stage('Test') {
            steps {
                sh "python -m pytest"
            }
        }
        stage('Package code') {
            steps {
                sh "zip -r python.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }
        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/pythonflask/python.zip  ${username}@${SERVER_IP}:/home/ec2-user/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ec2-user/python.zip -d /home/ec2-user/app/
                        python3 -m venv /home/ec2-user/app/venv --clear
                                source /home/ec2-user/app/venv/bin/activate
                                
                                # Install dependencies
                                pip install --upgrade pip
                                pip install -r /home/ec2-user/app/requirements.txt --no-cache-dir
                                
                                # Restart service
                                sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
       
        
       
        
    }
}
