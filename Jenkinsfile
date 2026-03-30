pipeline {
    agent { label 'agent-01' }

    options {
        gitLabConnection('homelab-gitlab')
        disableConcurrentBuilds()
    }

    environment {
        ANSIBLE_LXC = credentials('ANSIBLE_LXC')
        REMOTE_DIR  = "/tmp/proxmox-iac-ops-${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                updateGitlabCommitStatus name: 'jenkins', state: 'running'
                checkout scm
            }
        }

        stage('Deploy to Ansible LXC') {
            steps {
                sshagent(['ansible-deploy-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${ANSIBLE_LXC} "mkdir -p ${REMOTE_DIR}"
                        scp -o StrictHostKeyChecking=no -r . ${ANSIBLE_LXC}:${REMOTE_DIR}
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sshagent(['ansible-deploy-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${ANSIBLE_LXC} "
                            pip install --break-system-packages --quiet ansible ansible-lint &&
                            cd ${REMOTE_DIR} &&
                            ansible-galaxy collection install -r requirements.yml -p ./collections
                        "
                    '''
                }
            }
        }

        stage('Lint') {
            steps {
                sshagent(['ansible-deploy-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${ANSIBLE_LXC} "
                            cd ${REMOTE_DIR} &&
                            ansible-lint roles/ playbooks/
                        "
                    '''
                }
            }
        }

    }

    post {
        success {
            updateGitlabCommitStatus name: 'jenkins', state: 'success'
        }
        failure {
            updateGitlabCommitStatus name: 'jenkins', state: 'failed'
        }
        always {
            sshagent(['ansible-deploy-key']) {
                sh 'ssh -o StrictHostKeyChecking=no ${ANSIBLE_LXC} "rm -rf ${REMOTE_DIR}"'
            }
            cleanWs()
        }
    }
}