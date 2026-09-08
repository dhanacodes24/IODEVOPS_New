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
- [📂 Repository Structure](#-repository-structure)
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

## 📂 Repository Structure

<p align="center">
  <img src="https://img.shields.io/badge/📁_Repo-Ansible_+_Jenkins-2ECC71?style=for-the-badge&logoColor=white" />
  <img src="https://img.shields.io/badge/🔐_Role_Based-Autosys-9B59B6?style=for-the-badge&logoColor=white" />
  <img src="https://img.shields.io/badge/🌍_Multi_Env-UAT_|_PROD-F39C12?style=for-the-badge&logoColor=white" />
</p>

> 🧭 **Legend:** 🧩 Role definition · ⚙️ Config/vars · 🎬 Task logic · 📜 Playbook · 🌐 Inventory · 🔑 Env vars

```
🏢 IODEVOPS/
│
├── 🧩 ta-autosys-ansible-roles/              🔖  Ansible Galaxy Role (Bitbucket Repo)
│   │
│   ├── 🛠️  Jenkinsfile                        ⭐  Jenkins Pipeline definition — the star of the show!
│   │
│   ├── 📘 meta/
│   │   └── 🏷️  main.yml                       ℹ️   Role metadata (author, description, etc.)
│   │
│   ├── ⚙️  defaults/
│   │   └── 🎚️  main.yml                       🔧  Default variable values (safe fallbacks)
│   │
│   ├── 🧬 vars/
│   │   └── 📦 main.yml                       🧵  Role-level variables (override defaults)
│   │
│   ├── 🔔 handlers/
│   │   └── 🚫 main.yml                       😴  Handlers (none needed — quiet role!)
│   │
│   └── 🎬 tasks/
│       ├── 🚪 main.yml                       🧭  Entry point — delegates via tags
│       ├── ▶️  start.yml                      🟢  Start Autosys tasks
│       ├── ⏹️  stop.yml                       🔴  Stop Autosys tasks
│       └── 📊 status.yml                     🟡  Status check tasks
│
└── 🌐 ansible-common/
    │
    ├── 📜 playbooks/
    │   └── 🗂️  gen3/
    │       └── 🎯 autosys.yml                🪞  Playbook (mirrors webinterface.yml pattern)
    │
    └── 🌍 inventories/
        │
        ├── 🏭 gen3-voor-prod/                                🔴  PRODUCTION environment
        │   ├── 📥 autosys_prod_requirements.yml              🔗  Role source (git+https)
        │   └── 🖥️  inventory/static/
        │       ├── 🏘️  autosys.yml                            👥  Host groups (primary/shadow)
        │       └── 🔑 group_vars/
        │           └── 🗝️  autosys_vars/
        │               └── 🔒 prod_autosys.yml                🧾  Server vars (user, gid, uid, cmd)
        │
        └── 🧪 gen3-voor-uat/                                 🟡  UAT / staging environment
            ├── 📥 autosys_uat_requirements.yml                🔗  Role source (git+https)
            └── 🖥️  inventory/static/
                ├── 🏘️  autosys.yml                            👥  Host groups (primary/shadow)
                └── 🔑 group_vars/
                    └── 🗝️  autosys_vars/
                        └── 🔓 uat_autosys.yml                  🧾  Server vars (user, gid, uid, cmd)
```

| 🧭 Path | 🎯 Purpose |
|---|---|
| 🧩 `ta-autosys-ansible-roles/` | The reusable Ansible **role** + the **Jenkinsfile** that drives it |
| 🎬 `tasks/` | Task logic split by action tag: `start`, `stop`, `status` |
| 📜 `playbooks/gen3/autosys.yml` | The actual playbook Jenkins invokes |
| 🏭 `inventories/gen3-voor-prod/` | 🔴 Production hosts, role requirements & server-specific vars |
| 🧪 `inventories/gen3-voor-uat/` | 🟡 UAT hosts, role requirements & server-specific vars |

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

## 🧱 Stages Breakdown — Presentation Deep-Dive 🎤

---

### 1️⃣ 📥 Stage: SCM Checkout

<p>
  <img src="https://img.shields.io/badge/Stage_1-SCM_Checkout-3498DB?style=flat-square&logoColor=white" />
</p>

1. 🔗 This stage **connects to Bitbucket** and checks out the latest Ansible code.
2. 📦 Source repository: `ansible-common.git` (branch: `main`)
3. 🔑 Authenticated using the stored Jenkins credential `test-com-svcacct-gtasrepo`

> 💬 **In short:** *"First, Jenkins reaches out to Bitbucket and grabs the freshest copy of our Ansible codebase."*

---

### 2️⃣ 📦 Stage: Import Ansible Roles

<p>
  <img src="https://img.shields.io/badge/Stage_2-Import_Roles-9B59B6?style=flat-square&logoColor=white" />
</p>

1. ⬇️ We **download the required Ansible Galaxy roles** onto the Ansible host (the host is already provisioned/created).
2. 📄 The role source and version details live in:
   - `ansible-common/inventories/gen3-voor-prod/autosys_prod_requirements.yml` 🏭
3. 🌐 A proxy (`prod_https_proxy`) is exported before hitting Galaxy, since the host has restricted internet access.
4. 📋 Once installed, `ansible-galaxy role list` confirms what got pulled down.

> 💬 **In short:** *"Think of this as npm install / pip install — but for Ansible roles, pulled straight from the requirements file."*

📁 **Key file:**
> `ansible-common/inventories/gen3-voor-prod/autosys_prod_requirements.yml`

---

### 3️⃣ 🚀 Stage: Autosys Scheduler Action (Deploy Stage)

<p>
  <img src="https://img.shields.io/badge/Stage_3-Deploy_%7C_Autosys_Action-1ABC9C?style=flat-square&logoColor=white" />
</p>

By this point, the required Ansible role is already downloaded on the Ansible server. 🎯 This stage is where the **real action** happens.

#### 🔄 Connection Flow

```
🖥️  Jenkins Server  ──────▶  🧠  Ansible Server  ──────▶  🗓️  Autosys Server
                (SSH agent: 🔑 boks-gtaansible-ssh)
```

1. 🔐 We connect to the Autosys server using the SSH agent **`boks-gtaansible-ssh`**.
2. ▶️ Jenkins then triggers the playbook:
   📁 `ansible-common/playbooks/gen3/autosys-prod-uat.yml`

3. 📜 This playbook contains:
   - 🖥️ `hosts` definition
   - 🔍 `gather_facts`
   - 🧬 variable files
   - 🌐 proxy settings
   - and finally **calls the role** 👇

   ```yaml
   tasks:
     - name: Run Autosys Role
       ansible.builtin.include_role:
         name: gtas.autosys
       tags:
         - always
   ```

4. ➡️ Once the role is included, execution moves into the **Ansible role directory** (`ta-autosys-ansible-roles/`).

#### 🎯 Host Targeting

5. 🧾 The list of servers the task runs against is **limited** via the `inventoryLimit` variable set in the pipeline:
   ```groovy
   def inventoryLimit = params.environment == 'UAT' ? 'autosys_uat_hosts' : 'autosys_prod_hosts'
   ```
6. 🖥️ Server names for that limit group are maintained here:
   📁 `ansible-common/inventories/gen3-voor-prod/inventory/static/autosys.yml`
   > ✏️ *Add/remove server names here to control where the task actually runs.*

#### 🏷️ Tag-Based Task Selection

7. 🎛️ The `--tags ${params.autosys_action}` flag decides **which task actually executes**.
8. 📁 Tags are resolved inside:
   `ta-autosys-ansible-roles/tasks/main.yml`
   > 💡 Whatever action (`start` / `stop` / `status` / `restart`) is chosen in **Build with Parameters** on Jenkins, that matching tag is called from `main.yml`.

#### 👤 Ownership & Variables (No Hardcoding! 🚫)

9. 🔒 Owner, group, UID/GID, and the Linux command to run are **never hardcoded** in the playbook — they're pulled from environment-specific vars:
   📁 `ansible-common/inventories/gen3-voor-prod/inventory/static/group_vars/autosys_vars/prod_autosys.yml`
10. ✅ Our `start.yml` / `stop.yml` tasks read `user`, `gid`, `uid`, and `cmd` values from this file at runtime.

> 💬 **In short:** *"Everything environment-specific — hosts, users, permissions — lives in inventory vars, not the playbook. That's what makes one playbook safely reusable across UAT and PROD."* ✨

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
