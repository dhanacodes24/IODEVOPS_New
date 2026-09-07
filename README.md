







----------------------------------------
```
pipeline {
    agent { label 'py38-ansible-2-9' }

    environment {
        ANSIBLE_LOCAL_TEMP              = "${WORKSPACE}/.ansible/tmp"
        ANSIBLE_DISPLAY_SKIPPED_HOSTS   = "${params.ansible_verbosity == '' ? 'false' : 'true'}"
        ANSIBLE_CONFIG                  = "${WORKSPACE}/playbooks/gen3/ansible.cfg"
        ANSIBLE_ROLES_PATH              = "${WORKSPACE}/.ansible/roles"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '20', daysToKeepStr: '60'))
    }

    parameters {
        choice(
            name: 'environment',
            choices: ['UAT', 'PROD'],
            description: 'Select the target environment'
        )
        choice(
            name: 'autosys_action',
            choices: ['autosys_status', 'autosys_start', 'autosys_stop', 'autosys_restart'],
            description: 'Select action to perform on Autosys scheduler'
        )
        string(
            name: 'ansible_verbosity',
            defaultValue: '',
            description: '(Optional) Ansible verbosity flag e.g. -v, -vv, -vvv'
        )
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git(
                    url: 'https://bitbucket.test.dev/scm/iodevops/ansible-common.git',
                    branch: 'main',
                    credentialsId: 'test-com-svcacct-gtasrepo'
                )
            }
        }

        stage('Import Ansible Roles') {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'test-com-svcacct-gtasrepo')]) {
                    sh label: 'Import Ansible Roles', script: """
                        export https_proxy=\$prod_https_proxy
                        export http_proxy=\$prod_https_proxy
                        ansible-galaxy role install -c ${params.ansible_verbosity} \
                            -r inventories/gen3/autosys_requirements.yml
                        ansible-galaxy role list ${params.ansible_verbosity}
                    """
                }
            }
        }

        stage('Autosys Scheduler Action') {
            steps {
                script {
                    def inventoryLimit = params.environment == 'UAT' ? 'autosys_uat_hosts' : 'autosys_prod_hosts'
                    sshagent(['boks-gtaansible-ssh']) {
                        sh label: 'Run Ansible Playbook', script: """
                            ansible-playbook playbooks/gen3/autosys-prod-uat.yml \
                                -i inventories/gen3/inventory/static \
                                --limit ${inventoryLimit} \
                                ${params.ansible_verbosity} \
                                --tags ${params.autosys_action} \
                                --extra-vars "target_hosts=${inventoryLimit}"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
        }
        success {
            emailext(
                to: 'my_user@text.com, my_user2@text.com',
                subject: "SUCCESS: ${currentBuild.fullDisplayName} - AUTOSYS-${params.environment}-${params.autosys_action.toUpperCase()}",
                body: "Autosys action '${params.autosys_action}' completed successfully on ${params.environment}. Please find the console output attached.",
                attachLog: true
            )
            cleanWs()
        }
        failure {
            emailext(
                to: 'my_user@text.com, my_user2@text.com',
                subject: "FAILURE: ${currentBuild.fullDisplayName} - AUTOSYS-${params.environment}-${params.autosys_action.toUpperCase()}",
                body: "Autosys action '${params.autosys_action}' FAILED on ${params.environment}. Please find the console output attached.",
                attachLog: true
            )
            cleanWs()
        }
    }
}

```
