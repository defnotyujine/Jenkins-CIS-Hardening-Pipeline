pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOST_PREFIX', defaultValue: 'rhel_host', description: 'Prefix for host aliases e.g. rhel_host becomes rhel_host1, rhel_host2')
        string(name: 'TARGET_IPS', defaultValue: '', description: 'Comma-separated list of target IPs e.g. 192.168.1.10,192.168.1.11')
        choice(name: 'TAGS', choices: ['level1-server', 'level1-workstation', 'level1-server,level1-workstation'], description: 'CIS level tags to apply')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (params.TARGET_IPS == '') {
                        error('TARGET_IPS is required. Please provide at least one IP.')
                    }
                }
            }
        }

        stage('Install Collections') {
            steps {
                sh 'ansible-galaxy collection install -r requirements.yml'
            }
        }

        stage('Install Roles') {
            steps {
                sh 'ansible-galaxy role install -r requirements.yml -p playbooks/roles/ --force'
            }
        }

        stage('Generate Inventory') {
            steps {
                script {
                    def ips = params.TARGET_IPS.split(',')
                    def hosts = ips.eachWithIndex.collect { ip, i ->
                        "    ${params.TARGET_HOST_PREFIX}${i + 1}:\n      ansible_host: ${ip.trim()}"
                    }.join('\n')
                    writeFile file: 'sysconfig/inventory.yml', text: "all:\n  hosts:\n${hosts}\n"
                }
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
                        extras: "--vault-password-file /tmp/vault_pass.txt --tags \"${params.TAGS}\""
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
