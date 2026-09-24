pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/python -m pip install --upgrade pip
                    .venv/bin/python -m pip install -r requirements.txt
                    .venv/bin/ansible-galaxy collection install -r requirements.yml
                '''
            }
        }

        stage('Ansible Lint') {
            steps {
                sh '''
                    .venv/bin/ansible-lint ansible-role/ playbook.yml
                '''
            }
        }

        stage('Syntax Check') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ansible-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        .venv/bin/ansible-playbook \
                        -i aws_ec2.yml \
                        playbook.yml \
                        --syntax-check
                    '''
                }
            }
        }

        stage('Check Mode') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ansible-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        .venv/bin/ansible-playbook \
                        -i aws_ec2.yml \
                        playbook.yml \
                        --check
                    '''
                }
            }
        }
    }
}
