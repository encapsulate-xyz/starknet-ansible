# Ansible Playbook to deploy Starknet Node and Attestor Node

This Ansible playbook automates the deployment and configuration of Starknet node and attestor node . It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator and node keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves secret environment file from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`
For example:
`mainnet/starknet/encapsulate/attestor/starknet-attestor.secrets.env`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)


### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/starknet-ansible.git
cd starknet-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `testnet.yml` or `mainnet.yml` to update.

Example for mainnet.yml

```yaml
---
# maintains mainnet inventory grouped by project names
all:
  vars:
    env: mainnet
  children:
    starknet:
      hosts:
        attestor.starknet.mainnet.encapsulate.xyz:
          type: attestor
        node.starknet.mainnet.encapsulate.xyz:
          type: node
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.
- **`group_vars/vault.yml`**: Store secret variables, such as `jwtsecret`, in this file.
- There are role specific variables defined in each roles `default/main.yml` and `vars/main.yml`.

**Note**: Make sure to set the `VAULT_TOKEN` environment variable, as it enables logging in and fetching secrets from HashiCorp Vault.

Create a `group_vars/vault.yml` with your preferred ansible-vault password:

```
ansible-vault create group_vars/vault.yml
```

Example `group_vars/vault.yml`:
```
# maintains anything sensitive like api keys
vault:
  default:
    hcp:
      vault:
        url: "vault_url"
  mainnet:
    node:
      ethereum_url: 
  testnet:
```

### Usage

1. First, install the dependencies:

  ```
  ansible-galaxy role install -r roles/requirements.yml
  ansible-galaxy collection install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password.

3. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

3. Then run the playbook:

  ```bash
  ansible-playbook setup_node.yml -l node.starknet.mainnet.encapsulate.xyz
  ```

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [node.starknet.mainnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: node.starknet.mainnet.encapsulate.xyz",
        "Groups: ['starknet']",
        "Project: starknet",
        "Environment: mainnet",
        "Type: node",
        "Version: v0.17.0",
        "Username: starknet",
        "Service Name: starknet",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [node.starknet.mainnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [node.starknet.mainnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: node.starknet.mainnet.encapsulate.xyz",
        "Project: starknet",
        "Environment: mainnet",
        "Type: node"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [node.starknet.mainnet.encapsulate.xyz]
```
