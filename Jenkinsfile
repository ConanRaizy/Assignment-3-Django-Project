pipeline {
    agent any
    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "18.221.154.179"
        EC2_KEY = credentials('ec2-ssh-private-key')
        PROJECT_DIR = "/home/ubuntu/pythonprojects/Assignment-3-Django-Project"
    }
    //triggers {
    //    githubPush()
    //}

    stages {
        stage('Update Code on EC2') {
            steps {
                script {
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            sudo apt update -y
                            sudo apt install -y python3-venv python3-pip
                            cd ${PROJECT_DIR}
                            git pull origin main
                            python3 -m venv comp314
                            source comp314/bin/activate
                            pip install -r requirements.txt
                            python3 manage.py migrate
                            pkill -f "manage.py runserver" || true
                            nohup python3 manage.py runserver 0.0.0.0:8000 > /tmp/django.log 2>&1 &
                        '
                        """
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Code updated and app restarted successfully on EC2!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
