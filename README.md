
```


/IODEVOPS/
├── ta-autosys-ansible-roles/              ← Ansible Galaxy Role (Bitbucket Repo)
│   ├── Jenkinsfile                        ← Jenkins Pipeline definition
│   ├── meta/
│   │   └── main.yml                       ← Role metadata
│   ├── defaults/
│   │   └── main.yml                       ← Default variable values
│   ├── vars/
│   │   └── main.yml                       ← Role-level variables
│   ├── handlers/
│   │   └── main.yml                       ← Handlers (none needed)
│   └── tasks/
│       ├── main.yml                       ← Entry point, delegates via tags
│       ├── start.yml                      ← Start Autosys tasks
│       ├── stop.yml                       ← Stop Autosys tasks
│       └── status.yml                     ← Status check tasks
│
└── ansible-common/
    ├── playbooks/
    │   └── gen3/
    │       └── autosys.yml                ← Playbook (mirrors webinterface.yml pattern)
    └── inventories/
        ├── gen3-voor-prod/
        │   ├── autosys_prod_requirements.yml   ← Role source (git+https)
        │   └── inventory/static/
        │       ├── autosys.yml                 ← Host groups (primary/shadow)
        │       └── group_vars/
        │           └── autosys_vars/
        │               └── prod_autosys.yml    ← Server vars (user, gid, uid, cmd)
        └── gen3-voor-uat/
            ├── autosys_uat_requirements.yml    ← Role source (git+https)
            └── inventory/static/
                ├── autosys.yml                 ← Host groups (primary/shadow)
                └── group_vars/
                    └── autosys_vars/
                        └── uat_autosys.yml     ← Server vars (user, gid, uid, cmd)


```


----------------------------------------

# 🚀 Autosys Scheduler Jenkins Pipeline

![Jenkins](https://img.shields.io/badge/CI-Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Ansible](https://img.shields.io/badge/Automation-Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.8-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-2ECC71?style=for-the-badge)

> A Jenkins Declarative Pipeline that runs Ansible playbooks to **start / stop / restart / check status** of the Autosys scheduler across **UAT** and **PROD** environments.

---

## 📋 Table of Contents
- [🔎 Overview](#-overview)
- [🗺️ Pipeline Flow](#️-pipeline-flow)
- [⚙️ Configuration](#️-configuration)
- [🎛️ Parameters](#️-parameters)
- [🧱 Stages Breakdown](#-stages-breakdown)
- [📬 Post Actions](#-post-actions)
- [📄 Full Pipeline Script](#-full-pipeline-script)

---

## 🔎 Overview

| 🏷️ Attribute | 📌 Value |
|---|---|
| **Agent Label** | `py38-ansible-2-9` |
| **Trigger Type** | Manual (parameterized) |
| **Config Tool** | Ansible |
| **Target Systems** | Autosys Scheduler (UAT / PROD) |
| **Build Retention** | Last `20` builds or `60` days |
| **Notifications** | ✅ Success & ❌ Failure emails with logs |

---

## 🗺️ Pipeline Flow

<p align="center">
  <img src="https://img.shields.io/badge/🟢_START-2ECC71?style=for-the-badge&logoColor=white" />
  &nbsp;➡️&nbsp;
  <img src="https://img.shields.io/badge/📥_SCM_CHECKOUT-3498DB?style=for-the-badge&logoColor=white" />
  &nbsp;➡️&nbsp;
  <img src="https://img.shields.io/badge/📦_IMPORT_ROLES-9B59B6?style=for-the-badge&logoColor=white" />
  &nbsp;➡️&nbsp;
  <img src="https://img.shields.io/badge/🚀_RUN_PLAYBOOK-1ABC9C?style=for-the-badge&logoColor=white" />
  &nbsp;➡️&nbsp;
  <img src="https://img.shields.io/badge/📑_ARCHIVE_LOGS-34495E?style=for-the-badge&logoColor=white" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/✅_SUCCESS_→_email_+_cleanup-27AE60?style=for-the-badge&logoColor=white" />
  &nbsp;&nbsp;<b>or</b>&nbsp;&nbsp;
  <img src="https://img.shields.io/badge/❌_FAILURE_→_email_+_cleanup-E74C3C?style=for-the-badge&logoColor=white" />
</p>

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         🚀  AUTOSYS PIPELINE FLOW                        │
└──────────────────────────────────────────────────────────────────────────┘

   🟢 START
      │
      ▼
   📥 SCM CHECKOUT  ──────────────  pulls ansible-common.git (main)
      │
      ▼
   📦 IMPORT ANSIBLE ROLES  ─────  ansible-galaxy role install
      │
      ▼
   🎯 SELECT ENVIRONMENT
      │
      ├── UAT  ──▶  🚀 RUN PLAYBOOK  (autosys_uat_hosts)
      │
      └── PROD ──▶  🚀 RUN PLAYBOOK  (autosys_prod_hosts)
                        │
                        ▼
                  📑 ARCHIVE LOGS  (*.log)
                        │
                        ▼
                  ✅ BUILD RESULT?
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
     🎉 SUCCESS              ⚠️  FAILURE
     • send email             • send email
     • clean workspace        • clean workspace
             │                     │
             └──────────┬──────────┘
                         ▼
                     🏁 FINISH
```

---

## ⚙️ Configuration

| 🟦 Environment Variable | 💡 Purpose |
|---|---|
| `ANSIBLE_LOCAL_TEMP` | Sets Ansible's local temp directory inside the workspace |
| `ANSIBLE_DISPLAY_SKIPPED_HOSTS` | Toggles display of skipped hosts based on verbosity flag |
| `ANSIBLE_CONFIG` | Points to the project's `ansible.cfg` |
| `ANSIBLE_ROLES_PATH` | Defines where Ansible Galaxy roles are installed |

---

## 🎛️ Parameters

| 🎚️ Parameter | 🧩 Type | 📥 Choices / Default | 📝 Description |
|---|---|---|---|
| `environment` | `choice` | `UAT`, `PROD` | Target environment for the action |
| `autosys_action` | `choice` | `autosys_status`, `autosys_start`, `autosys_stop`, `autosys_restart` | Action performed on the Autosys scheduler |
| `ansible_verbosity` | `string` | *(empty)* | Optional Ansible verbosity flag (`-v`, `-vv`, `-vvv`) |

---

## 🧱 Stages Breakdown

### 1️⃣ 📥 SCM Checkout
> 🔵 **Blue = Source Control**
Pulls the latest Ansible code from the `main` branch of the `ansible-common` Bitbucket repository.

### 2️⃣ 📦 Import Ansible Roles
> 🟣 **Purple = Dependency Management**
Installs required roles via `ansible-galaxy` using proxy credentials, then lists installed roles.

### 3️⃣ 🚀 Autosys Scheduler Action
> 🟢 **Teal = Execution**
Runs `autosys-prod-uat.yml` against the correct inventory (`autosys_uat_hosts` or `autosys_prod_hosts`) based on the selected environment and action tag.

---

## 📬 Post Actions

| 🟩 Condition | 📧 Action |
|---|---|
| **Always** | 📑 Archives all `*.log` files (empty archive allowed) |
| **✅ Success** | Sends success email with logs → 🧹 cleans workspace |
| **❌ Failure** | Sends failure email with logs → 🧹 cleans workspace |

---

## 📄 Full Pipeline Script

```groovy
pipeline {
    agent { label 'py38-ansible-2-9' }  
    // 🖥️ Agent: Runs pipeline on node labeled "py38-ansible-2-9"
    // 📌 Summary: Ensures all steps execute on a machine with Python 3.8 + Ansible 2.9 installed

    environment {
        ANSIBLE_LOCAL_TEMP            = "${WORKSPACE}/.ansible/tmp"  
        ANSIBLE_DISPLAY_SKIPPED_HOSTS = "${params.ansible_verbosity == '' ? 'false' : 'true'}"  
        ANSIBLE_CONFIG                = "${WORKSPACE}/playbooks/gen3/ansible.cfg"  
        ANSIBLE_ROLES_PATH            = "${WORKSPACE}/.ansible/roles"  
    }
    // 🌍 Summary: Defines Ansible paths, config, and verbosity behavior for consistent execution

    options {
        buildDiscarder(logRotator(numToKeepStr: '20', daysToKeepStr: '60'))  
    }
    // 🧹 Summary: Keeps last 20 builds or 60 days of history, discards older ones to save space

    parameters {
        choice(name: 'environment', choices: ['UAT', 'PROD'], description: 'Select the target environment')
        choice(name: 'autosys_action', choices: ['autosys_status', 'autosys_start', 'autosys_stop', 'autosys_restart'], description: 'Select action to perform on Autosys scheduler')
        string(name: 'ansible_verbosity', defaultValue: '', description: '(Optional) Ansible verbosity flag e.g. -v, -vv, -vvv')
    }
    // 🎛️ Summary: Provides user inputs (environment, action, verbosity) to customize pipeline run

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
        // 📥 Summary: Pulls latest Ansible code from Bitbucket repo (main branch)

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
        // 📦 Summary: Installs required Ansible roles via Galaxy and lists them (using proxy if needed)

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
        // 🚀 Summary: Runs Ansible playbook on Autosys hosts (UAT/PROD) with chosen action (status/start/stop/restart)
    }

    post {
        always {
            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
        }
        // 📑 Summary: Archives logs for every build, regardless of result

        success {
            emailext(
                to: 'my_user@text.com, my_user2@text.com',
                subject: "SUCCESS: ${currentBuild.fullDisplayName} - AUTOSYS-${params.environment}-${params.autosys_action.toUpperCase()}",
                body: "✅ Autosys action '${params.autosys_action}' completed successfully on ${params.environment}. Logs attached.",
                attachLog: true
            )
            cleanWs()
        }
        // 🎉 Summary: Sends success email with logs + cleans workspace

        failure {
            emailext(
                to: 'my_user@text.com, my_user2@text.com',
                subject: "FAILURE: ${currentBuild.fullDisplayName} - AUTOSYS-${params.environment}-${params.autosys_action.toUpperCase()}",
                body: "❌ Autosys action '${params.autosys_action}' FAILED on ${params.environment}. Logs attached.",
                attachLog: true
            )
            cleanWs()
        }
        // ⚠️ Summary: Sends failure email with logs + cleans workspace
    }
}
```

---

<p align="center">
  <sub>🔧 Maintained via Jenkins Declarative Pipeline • Powered by Ansible 🐘</sub>
</p>
