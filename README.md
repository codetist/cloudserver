## Hetzner Cloud Server

Create and deploy personal cloud servers through 
[Hetzner Cloud](https://www.hetzner.com/de/cloud/).
My personal setup, maybe it could become useful for somebody else. :)

### Project setup

#### Vault

* Create `.vault_password_file` in project root, containing
  the password for your new vault
* Create vault: `ansible-vault create group_vars/cloudservers/vault.yml`
* Edit vault: `ansible-vault edit group_vars/cloudservers/vault.yml`, required
  variables can be found in `vault variables` section of 
  [main.yml](./group_vars/cloudservers/main.yml) 

#### Dependencies

Ansible Galaxy: 
```shell
ansible-galaxy install -r requirements.yml --force
```

Virtual Python Environment (might be necessary for `homebrew` users on macOS)
```shell
python3 -m venv ~/.pyansibleenv
source ~/.pyansibleenv/bin/activate
python3 -m pip install requests
python3 -m pip install python-dateutil
```

#### Configure cloud server(s)

Modify the [inventory](./hosts/inventory.yml) to match your desired cloud server product and
configuration.

### Run playbooks

```shell
 ## enable python venv if necessary
 source ~/.pyansibleenv/bin/activate
 
 ## run playbooks
 ansible-playbook <playbookname>.yml  
```