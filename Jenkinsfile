pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOST', defaultValue: 'rhel_host', description: 'Alias for the target host')
        string(name: 'TARGET_IP', defaultValue: '', description: 'IP of target server')
    }

    stages {
        stage('Install Collections') {
            steps {
                sh 'ansible-galaxy collection install -r requirements.yml'
            }
        }

        stage('Install Roles') {
            steps {
                sh 'ansible-galaxy role install -r requirements.yml -p playbooks/roles/'
            }
        }

        stage('Generate Inventory') {
            steps {
                sh """
cat > sysconfig/inventory.yml <<EOF
all:
  hosts:
    ${params.TARGET_HOST}:
      ansible_host: ${params.TARGET_IP}
EOF
                """
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'ansible-vault-pass',
                        variable: 'VAULT_PASS'
                    )
                ]) {
                    sh 'echo $VAULT_PASS > /tmp/vault_pass.txt'
                    ansiblePlaybook(
                        playbook: 'playbooks/main_playbook.yml',
                        inventory: 'sysconfig/inventory.yml',
                        extras: '--vault-password-file /tmp/vault_pass.txt'
                    )
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f /tmp/vault_pass.txt'
        }
    }
}
