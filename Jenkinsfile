pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOST', defaultValue: 'rhel_host', description: 'Alias for the target host')
        string(name: 'TARGET_IP', defaultValue: '', description: 'IP of target server')
    }

    stages {
        stage('Install Collections') {
            steps {
                sh 'ansible-galaxy collection install collections/requirements.yml'
            }
        }

        stage('Install Roles') {
            steps {
                sh 'ansible-galaxy install -r requirements.yml -p playbooks/roles/'
            }
        }
    }
}