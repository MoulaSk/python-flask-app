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
                sh '''
                rm -f myapp.zip
                zip -r myapp.zip . -x "venv/*" "*.git*" "*.pyc" "__pycache__/*"
                '''

            }
        }
stage('Deploy to Prod') {
    steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'MY_SSH_KEY', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
            sh '''
            scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
            ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                rm -rf /home/ec2-user/app
                mkdir -p /home/ec2-user/app
                unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                python3 -m venv /home/ec2-user/app/venv
                /home/ec2-user/app/venv/bin/pip install --upgrade pip
                /home/ec2-user/app/venv/bin/pip install -r /home/ec2-user/app/requirements.txt
                sudo cp /home/ec2-user/app/flask.service /etc/systemd/system/flaskapp.service
                sudo systemctl daemon-reexec
                sudo systemctl daemon-reload
                sudo systemctl restart flaskapp.service
            EOF
            '''
                }
            }
        }
    }
}
