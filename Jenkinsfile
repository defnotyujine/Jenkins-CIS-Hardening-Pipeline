pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOST_PREFIX', defaultValue: 'rhel_host', description: 'Prefix for host aliases e.g. rhel_host becomes rhel_host1, rhel_host2')
        string(name: 'VM_COUNT', defaultValue: '2', description: 'Number of VMs to provision')
        choice(name: 'TAGS', choices: ['', 'level1-server', 'level1-workstation', 'level1-server,level1-workstation'], description: 'CIS level tags to apply (leave blank to run all tasks)')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (params.VM_COUNT == '') {
                        error('VM_COUNT is required.')
                    }
                }
            }
        }

        stage('Checkout Terraform') {
            steps {
                dir('terraform') {
                    git branch: 'jenkins-terraform-integration',
                        url: 'https://github.com/defnotyujine/Terraform-KVM.git'
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

        stage('Terraform Apply') {
            steps {
                sh """
                    cd terraform/
                    terraform init
                    terraform apply -auto-approve -var="vm_count=${params.VM_COUNT}"
                """
            }
        }

        stage('Generate Inventory') {
            steps {
                script {
                    def ips = sh(
                        script: "cd terraform/ && terraform output -json default-vm_ips | python3 -c \"import sys,json; print(','.join(json.load(sys.stdin)))\"",
                        returnStdout: true
                    ).trim()

                    def ipList = ips.split(',')
                    def hosts = ""
                    for (int i = 0; i < ipList.size(); i++) {
                        hosts += "    ${params.TARGET_HOST_PREFIX}${i + 1}:\n      ansible_host: ${ipList[i].trim()}\n"
                    }
                    writeFile file: 'sysconfig/inventory.yml', text: "all:\n  hosts:\n${hosts}"
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
                    script {
                        def extraArgs = "--vault-password-file /tmp/vault_pass.txt"
                        if (params.TAGS) {
                            extraArgs += " --tags \"${params.TAGS}\""
                        }
                        sh 'echo $VAULT_PASS > /tmp/vault_pass.txt'
                        ansiblePlaybook(
                            playbook: 'playbooks/main_playbook.yml',
                            inventory: 'sysconfig/inventory.yml',
                            extras: extraArgs
                        )
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f /tmp/vault_pass.txt'
        }
        failure {
            sh '''
                cd terraform/
                terraform destroy -auto-approve
            '''
        }
    }
}
