# Install Ansible

```sh
pip install ansible
```
-------------------
# Install AppDynamics Collection
Currently there is an issue where running `ansible-galaxy collections install appdynamics.agents` installs the oldest version of the collection not the newest. So explicitly stating the latest version is necessary. 
```sh
ansible-galaxy collection install appdynamics.agents:23.4.2-867
```
make get-agent.sh executable
```sh
chmod a+x .ansible/collections/ansible_collections/appdynamics/agents/roles/common/files/get-agent.sh
```
# Create Role/Playbook for Agent Deployment
### Role:
1) Create a role with the `ansible-galaxy` command inside the roles directory for ansible
```sh
ansible-galaxy init appdynamics-agents
```
2) Update `vars/main.yml` with values for the AppDynamics Role. [AppDynamics Ansible Documentation](https://docs.appdynamics.com/appd/23.x/latest/en/application-monitoring/install-app-server-agents/agent-management/standalone-host-platforms/ansible) (Example for cSaaS controller)
```yml
---
controller_account_name: "<Customer Account Name>"
controller_host_name: "{{ controller_account_name }}.saas.appdynamics.com"
enable_ssl: true
controller_port: 443
```
3) Add access key to `vars/main.yml` with `ansible-vault` command. [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/vault_guide/vault_encrypting_content.html#creating-encrypted-variables) (Below command will copy vault string into vars file from role root directory)
```sh
ansible-vault encrypt_string <password_source> '<access key>' --name 'controller_account_access_key' >> vars/main.yml
```
4) The `defaults/main.yml` file can be used to store default version numbers for agents
```yml
---
agent_version: 23.6.0
java_application_name: 'AD-Ecommerce'
java_tier_name: 'Ecommerce-Services'
sim_enabled: false
enable_analytics_agent: false
linux_install_root: './AppDynamics' #here for demonstration purposes. defualt root is /opt/appdynamics
```
5) Create the tasks inside `tasks/main.yml`
```yml
---
# tasks file for agentlab
- name: Include variables for the controller settings
  # Include all yaml files under the vars directory
  include_vars:
    dir: vars
    extensions:
      - 'yaml'
      - 'yml'
- include_role:
    name: appdynamics.agents.java
  vars:
    agent_type: java
    application_name: "{{ java_application_name }}" # agent default application
    tier_name: "{{ java_tier_name }}" # agent default tier
- include_role:
    name: appdynamics.agents.machine
  vars:
    agent_type: machine
```
### Playbook:
1) Create a Playbook file. Example: `appdynamics-setup.yml`
```yml
---
- name: install appdynamics agents
  hosts: localhost
  roles:
    - appdynamics-agents
```
2) Run ansible playbook
```sh
ansible-playbook <ansible-vault_password_source> appdynamics-setup.yml 
```
# Notes
- Creating an Ansible Role is not necessary. A simple playbook is fine. The Role makes it more useful for the customer to drop it into existing playbooks as needed
- A Champion writing Ansible playbooks will have a better idea of where everything fits inside of their environment
- For POV purposes, a simple playbook can be created using examples from the [AppDynamics Ansible Documentation](https://docs.appdynamics.com/appd/23.x/latest/en/application-monitoring/install-app-server-agents/agent-management/standalone-host-platforms/ansible)

# Cloud Academy "Lab"
We can make use of the [Getting Started with Ansible](https://cloudacademy.com/lab/getting-started-ansible/) in Cloud Academy to quickly set up an environment
1) Start Lab
2) Log in to AWS
3) Connect to EC2 instance
4) `sudo apt upgrade`
5) `sudo apt install pip jq`
6) `pip install --upgrade ansible`
7) `mkdir roles`
8) `cd roles`
9)  Run steps above starting at [Install AppDynamics Collection](#install-appdynamics-collection)
