## Hetzner Cloud Server

Create and deploy personal cloud servers through 
[Hetzner Cloud](https://www.hetzner.com/de/cloud/).
A personal setup, maybe it could become useful for somebody else. :)
Most of the project is not specific to Hetzner. But it makes use
of the Hetzner API to manage servers, SSH keys and DNS configuration.

### References

* [Hetzner API](https://docs.hetzner.cloud/)
* [hetzner.hcloud on Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/hetzner/hcloud/docs/) 
* [community.dns on Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/community/dns/docs/)

### Project setup

#### Vault

* Create `.vault_password_file` in project root, containing
  the password for your new vault
* Create vault: `ansible-vault create group_vars/cloudservers/vault.yml`
* Edit vault: `ansible-vault edit group_vars/cloudservers/vault.yml`, required
  variables can be found in `vault variables` section of 
  [main.yml](./group_vars/cloudservers/main.yml). All values that originate 
  from the vault are postfixed with `_vault`.

#### Dependencies

##### Required: Install Ansible Galaxy packages
```shell
ansible-galaxy install -r requirements.yml --force
```

##### Optional: Virtual Python Environment
Virtual Python Environment (might be necessary for `homebrew` users on macOS, 
as for example `python-requests` and `python-dateutil` packages were disabled 
on homebrew currently).
```shell
python3 -m venv ~/.pyansibleenv
source ~/.pyansibleenv/bin/activate
python3 -m pip install requests
python3 -m pip install passlib
python3 -m pip install python-dateutil
python3 -m pip install ansible
```

#### Configure cloud server(s)

Modify the [inventory](./hosts/inventory.yml) to match your desired cloud server
product and configuration.

### Playbooks overview

#### 01_init_cloudserver.yml

Creates a new server on Hetzner cloud, if it does not already exist, and
implements basic security settings:

* create server and DNS records
* update/upgrade packages and reboot
* enable unattended upgrades
* setup iptables
* secure SSH: root and password login not allowed
* setup Wireguard and print generated client configs (yaml and QR)
* SSH is only allowed via Wireguard VPN, no ssh access on public network interface

### Run playbooks

```shell
 ## enable python venv if necessary
 source ~/.pyansibleenv/bin/activate
 
 ## run playbooks
 ansible-playbook <playbookname>.yml  
```

### Hints, tipps and tricks

#### Hetzner web console

In case you locked yourself out of your server, Hetzner provides a web console
to connect to the servers internally. Use the `deploy_user` and specified password.
Credentials are created during server creation using a 
[cloud-init template](./roles/secured_cloudserver/templates/conf/cloud-init).
Keyboard layout may be English when typing passwords.

#### Hetzner DNS configuration

The current implementation uses subdomains that get assigned to the cloud servers
and the FQDN is used for SSH connections. Newly added subdomain DNS entries show 
up very quickly, but it might still be too slow for the playbooks. Causing the
`01_init_cloudserver.yml` playbook to fail after server creation. In this case
just wait a few minutes until subdomains get pingable and restart the playbook.

#### Wireguard vpn

* The [01_init_cloudserver.yml](./01_init_cloudserver.yml) sets up all servers 
  including Wireguard and disables public SSH access. Make sure to always have at
  least one Wireguard client configured.

* You can use `ansible-playbook 01_init_cloudserver.yml -t wgclients` to print
  all client configs (plain text config files and QR codes).

* Playbooks will try to connect via VPN first and fallback to public SSH, if VPN
  connection is not available.

* A single Wireguard client config might be shared on multiple devices, as long they
  are not used at the same time.

* For the sake of simplicity the same server and client keys are used on all
  hosts defined in the [main.yml](./group_vars/cloudservers/main.yml). This works
  fine, as long each server uses a different endpoint.

#### Multiple Wireguard connections on macOS

MacOS allows the creation of multiple vpn connections and also to enable them
simultaneously. But this leads to only one working connection at the same time
silently. This has been discussed on Apple Developer forums:
https://forums.developer.apple.com/forums/thread/687371

> You will find that on both macOS and iOS as soon as you start another instance of
> a NEPacketTunnelProvider it will take precedence over the previously running 
> NEPacketTunnelProvider instance.

As the official [Wireguard app](https://apps.apple.com/de/app/wireguard/id1451685025) 
uses the macOS network stack, it can only be used for a single connection.
For multiple simultaneous connections use Wireguard via [Homebrew](https://brew.sh). 

```shell
# Install Wireguard
brew install wireguard-tools
```

This way you can use more than one Wireguard connection at a time:

```shell
# Copy Wireguard client config from the playbooks output
# to a arbitrary location
nano /Users/myuser/Downloads/wg0.conf
nano /Users/myuser/Downloads/wg1.conf

# Start tunnels
sudo wg-quick up /Users/myuser/Downloads/wg0.conf
sudo wg-quick up /Users/myuser/Downloads/wg1.conf

# Stop tunnels
sudo wg-quick down /Users/myuser/Downloads/wg0.conf
sudo wg-quick down /Users/myuser/Downloads/wg1.conf
```

alternatively 

```shell
# Copy Wireguard to /opt/homebrew/etc/wireguard
nano /opt/homebrew/etc/wireguard/wg0.conf
nano /opt/homebrew/etc/wireguard/wg1.conf

# Start tunnels
sudo wg-quick up wg0
sudo wg-quick up wg1

# Stop tunnels
sudo wg-quick down wg0
sudo wg-quick down wg1
```