pipeline {
    agent any
    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "3.21.168.219"
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
                        ssh -o StrictHostKeyChecking=no -o ServerAliveInterval=30 -o ConnectTimeout=30 ${EC2_USER}@${EC2_HOST} '
                            set -e

                            cd ${PROJECT_DIR}

                            # Sync with GitHub (no merge conflicts)
                            git fetch origin
                            git reset --hard origin/main

                            # Create venv if it does not already exist
                            if [ ! -d "comp314" ]; then
                                python3 -m venv comp314
                            fi

                            # Activate venv, install deps, migrate — all in one subshell
                            (
                                source comp314/bin/activate
                                pip install --quiet -r requirements.txt
                                python3 manage.py migrate --noinput
                            )

                            # Stop any existing runserver
                            pkill -f "manage.py runserver" || true
                            sleep 1

                            # Start Django in the background, venv activated inside nohup subshell
                            nohup bash -c "source ${PROJECT_DIR}/comp314/bin/activate && python3 manage.py runserver 0.0.0.0:8000" > /tmp/django.log 2>&1 &

                            echo "Deployment done. Django is starting..."
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