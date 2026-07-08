# Pi-hole on Oracle Linux 9 (Ansible)

Deploys [Pi-hole](https://pi-hole.net/) to two Oracle Linux 9 servers using the
official unattended installer.

## Layout

```
ansible.cfg
inventory.ini            # the two target hosts
site.yml                 # top-level playbook
group_vars/pihole.yml    # user-tunable variables
roles/pihole/            # install + configure role
```

## Prerequisites

- Ansible 2.14+ on the control node.
- Required collection:
  ```bash
  ansible-galaxy collection install ansible.posix
  ```
- SSH access to both hosts with a user that can `sudo`.

## Configure

1. Edit [inventory.ini](inventory.ini) with your hosts' IPs and SSH user.
2. Edit [group_vars/pihole.yml](group_vars/pihole.yml):
   - `pihole_dns_upstream` — upstream resolvers.
   - `pihole_web_password` — **change this**, ideally with Ansible Vault.

### Protect the web password with Vault (recommended)

```bash
ansible-vault encrypt_string 'YourStrongPassword' --name 'pihole_web_password'
```

Paste the output into `group_vars/pihole.yml`, then run with `--ask-vault-pass`.

## Run

```bash
# Check connectivity
ansible pihole -m ping

# Dry run
ansible-playbook site.yml --check

# Deploy
ansible-playbook site.yml
```

Override the password inline instead of editing files:

```bash
ansible-playbook site.yml -e "pihole_web_password=YourStrongPassword"
```

## After install

- Admin UI: `http://<server-ip>/admin`
- Point client DNS at each server's IP.

## Notes

- Pi-hole does not officially support Oracle Linux, so `PIHOLE_SKIP_OS_CHECK`
  is set. The install works on the RHEL-compatible base but is unsupported
  upstream.
- Firewall rules (DNS + TCP/80) are opened automatically when firewalld is
  active.
- The role is idempotent: it skips installation if Pi-hole is already present
  and only re-applies configuration.
