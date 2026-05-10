# Jenkins CIS Hardening Pipeline

Automated CIS benchmark hardening for RHEL 8, 9, and 10 using Jenkins and Ansible Lockdown roles.

## Requirements

- Jenkins server with Ansible plugin and `ansible-core` + `sshpass` installed
- Jenkins credential with ID `ansible-vault-pass` storing the vault password
- Target hosts running RHEL 8, 9, or 10 with SSH access and sudo privileges

## Usage

Trigger the pipeline via **Build with Parameters**:

- `TARGET_HOST_PREFIX` - alias prefix for target hosts e.g. `rhel_host`
- `TARGET_IPS` - comma-separated IPs e.g. `192.168.1.10,192.168.1.11`
- `TAGS` - CIS level to apply, leave blank to run all tasks

## Vault

Secrets are stored encrypted in `sysconfig/group_vars/all/secrets.yml`.

```bash
ansible-vault edit sysconfig/group_vars/all/secrets.yml
```