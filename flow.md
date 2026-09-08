# 🎤 Autosys Pipeline — Presentation Notes

---

## 1️⃣ 📥 SCM Checkout

```groovy
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
```

- 🔗 This stage **connects to Bitbucket**
- 📦 Pulls the latest Ansible code from the repo

---

## 2️⃣ 📦 Import Ansible Roles

```groovy
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
```

- ⬇️ We **download the Ansible Galaxy roles** onto the Ansible host *(host is already created)*
- 📄 Role details & path are defined in:
  `autosys_prod_requirements.yml`
- 📁 Full path:
  `ansible-common/inventories/gen3-voor-prod/autosys_prod_requirements.yml`

---

## 3️⃣ 🚀 Deploy Stage — Autosys Scheduler Action

```groovy
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
```

- ✅ At this point, the Ansible role is already downloaded on the Ansible server
- 🔑 We use the agent **`boks-gtaansible-ssh`** to connect to the Autosys server
- 🔄 **Flow:** Ansible Server ➡️ Jenkins Server ➡️ Autosys Server

- ▶️ Next, we go to the playbook:
  `ansible-common/playbooks/gen3/autosys-prod-uat.yml`

- 📜 This playbook contains `hosts`, `gather_facts`, var files, proxy settings — and from here it **calls the Role**:

  ```yaml
  tasks:
    - name: Run Autosys Role
      ansible.builtin.include_role:
        name: gtas.autosys
      tags:
        - always
  ```

- ➡️ Once roles are downloaded, it goes to the **Ansible role directory**

- 🎯 The list of servers the task runs on is **limited** — this limit is defined in the pipeline as `inventoryLimit`
- 📁 `ansible-common/inventories/gen3-voor-prod/inventory/static/autosys.yml`
  > Under this limit, we add the server names where the task should run

- 🏷️ Next, `--tags ${params.autosys_action}`
- 📁 `ta-autosys-ansible-roles/tasks/main.yml`
  > Whatever tag we give in **Build with Parameters** in the Jenkins pipeline, that tag gets called from `main.yml`

- 👤 Regarding **owner and group** — we define these in `ansible-common`, **not hardcoded** in the playbook:
- 📁 `ansible-common/inventories/gen3-voor-prod/inventory/static/group_vars/autosys_vars/prod_autosys.yml`
  > Our `start` / `stop` playbooks take **owner, gid, uid, and Linux command** from this file

---










---------

---------

------

Here’s your **clean, GitHub‑friendly rewrite** of the pipeline stages with corrected spelling, consistent indentation, and clear comments summarizing each stage’s purpose and flow.  
I’ve formatted it for readability and documentation clarity — perfect for version control or team review 👇  

---

```groovy
// ============================================================
// 🧩 Jenkins Pipeline: Autosys Deployment Workflow
// ============================================================

stages {

    // ========================================================
    // 🧾 STAGE 1: SCM CHECKOUT
    // ========================================================
    stage('SCM Checkout') {
        steps {
            git(
                url: 'https://bitbucket.test.dev/scm/iodevops/ansible-common.git',
                branch: 'main',
                credentialsId: 'test-com-svcacct-gtasrepo'
            )
        }
    }
    // 📥 Summary: Connects to Bitbucket and pulls the latest Ansible code from the main branch.
    // 🧠 Purpose: Ensures Jenkins has the most recent version of playbooks and inventory files.


    // ========================================================
    // 📦 STAGE 2: IMPORT ANSIBLE ROLES
    // ========================================================
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
    // 📦 Summary: Downloads Ansible Galaxy roles on the Ansible host.
    // 📁 Source File: ansible-common/inventories/gen3-voor-prod/autosys_prod_requirements.yml
    // 🧠 Purpose: This file defines the roles and their paths required for Autosys deployment.


    // ========================================================
    // 🚀 STAGE 3: AUTOSYS SCHEDULER ACTION (DEPLOY)
    // ========================================================
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
    // 🚀 Summary: Executes the Autosys playbook on target hosts (UAT/PROD) using SSH agent.
    // 🔗 Flow: Jenkins → Ansible Server → Autosys Server
    // 📁 Playbook Path: ansible-common/playbooks/gen3/autosys-prod-uat.yml
    // 🧠 Purpose: This playbook defines hosts, gather facts, vars, proxy, and calls the role `gtas.autosys`.

    // Example from playbook:
    // tasks:
    //   - name: Run Autosys Role
    //     ansible.builtin.include_role:
    //       name: gtas.autosys
    //     tags:
    //       - always

    // 📂 Role Directory: ansible-common/inventories/gen3-voor-prod/inventory/static/autosys.yml
    // 🧩 Limit: The list of servers where tasks run is defined by `inventoryLimit`.
    // 🏷️ Tags: The tag passed in Jenkins parameters triggers the corresponding task in `ta-autosys-ansible-roles/tasks/main.yml`.

    // ⚙️ Owner/Group Configuration:
    // Defined in:
    // ansible-common/inventories/gen3-voor-prod/inventory/static/group_vars/autosys_vars/prod_autosys.yml
    // 🧠 Purpose: Playbooks (start/stop) fetch owner, GID, UID, and Linux commands from this file — not hardcoded.
}
```

---

### ✅ Highlights for GitHub Documentation
| 🧩 Stage | 📝 Description | 📁 Key Files |
|-----------|----------------|--------------|
| **SCM Checkout** | Pulls latest Ansible code from Bitbucket | `ansible-common.git` |
| **Import Ansible Roles** | Installs required roles from Galaxy | `autosys_prod_requirements.yml` |
| **Autosys Scheduler Action** | Runs playbook on Autosys servers | `autosys-prod-uat.yml`, `autosys.yml`, `prod_autosys.yml` |

---

Would you like me to add a **README.md section** (with emojis + markdown table) summarizing this pipeline for your GitHub repo? It’ll look professional and help teammates understand the workflow instantly.












-------------



------------



—----------



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

      

This stage connects to bitbucket 
—-----------------
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

- We download ansible galaxy roles on ansible hosts which we have already created 
Autosys_prod_requirements.yml this file has details and path of roles
ansible-common/inventories/gen3-voor-prod


ansible-common/inventories/gen3-voor-prod/autosys_prod_requirements.yml








DEPLOY STAGE 

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


—- 
—
At this stage we have details of ansible role downloaded on ansible server
Boks-gtaansible-ssh with this agent we connect to autosys server the flow is Ansible server - Jenkins  server- Autosys server
Then we Go to    ansible-playbook playbooks/gen3/autosys-prod-uat.yml 
ansible-common/playbooks/gen3/autosys-prod-uat.yml 


         This playbook contains host , gatherfacts and var files , proxy and from this playbook (/autosys-prod-uat.yml )it calls Role

  tasks:
    - name: Run Autosys Role
      ansible.builtin.include_role:
        name: gtas.autosys
      tags:
        - always


→ Once roles are downloaded It goes to ansible role directory 
List of servers on which task will be run is limited this  limit defined in pipeline inventoryLimit → 


ansible-common/inventories/gen3-voor-prod/inventory/static/autosys.yml

Under this limit we had  to added server names where tasks should be run 

– next tags ${params.autosys_action} \
– ta-autosys-ansible-roles/tasks/main.yml

Whatever tag we give in build with parameters in jenkins pipeline that tag will be called from main.yml

– Regarding owner and group we define in ansible common do not hardcode in playbook:
ansible-common/inventories/gen3-voor-prod/inventory/static/group_vars/autosys_vars/prod_autosys.yml

Our playbooks start stop will take owner , gid , uid , linux  command from this file

