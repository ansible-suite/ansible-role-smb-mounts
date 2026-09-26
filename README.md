# Ansible Role: smb_mounts

- [Ansible Role: smb\_mounts](#ansible-role-smb_mounts)
  - [Requirements](#requirements)
  - [Role Variables](#role-variables)
  - [Example `requirements.yml` for Ansible site](#example-requirementsyml-for-ansible-site)
  - [Generate password hash for Ansible Vault](#generate-password-hash-for-ansible-vault)
  - [Example Playbook](#example-playbook)
  - [Example `group_vars`](#example-group_vars)
    - [For Server-like](#for-server-like)
    - [For Workstation](#for-workstation)

## Requirements

This role requires the `ansible.posix` collection for the mount module.

```bash
ansible-galaxy collection install ansible.posix
```

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml` or variables definitions):

- `smb_shares`: List of SMB shares to mount. Each item can contain:
  - `name`: Unique name for the share and credentials file.
  - `mount_point`: Local directory path where the share will be mounted.
  - `share_path`: UNC path to the SMB share (e.g. `//server/share`).
  - `username`: Username for SMB authentication.
  - `password`: Password or Ansible Vault encrypted string for SMB authentication.
  - `domain`: Optional Windows domain name.
  - `uid`: Owner UID/username for the mount options (defaults to `'0'`).
  - `gid`: Group GID/groupname for the mount options and optional group creation (defaults to `'0'`).
  - `allowed_users`: Optional list of local system users to be added to the specified `gid` group.


## Example `requirements.yml` for Ansible site

```yaml
roles:
  - name: ansible-suite.smb_mounts
    src: git+https://github.com/ansible-suite/ansible-role-smb-mounts.git
```

## Generate password hash for Ansible Vault

```bash
ansible-vault encrypt_string 'SecretPassword123' --name 'password'
```

Generated output can be used in the `password` field of the `smb_shares` variable.

## Example Playbook

```yaml
- hosts: all
  vars:
    smb_shares:
      - name: "backup_nas"
        mount_point: "/mnt/backup"
        share_path: "//192.168.1.100/Backup"
        username: "backup_user"
        password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          36393661...
  roles:
    - ansible-suite.smb_mounts
```

## Example `group_vars`

### For Server-like

This example is for a server-like environment where the SMB shares are mounted with specific permissions and access controls.

```yaml
smb_mount_mode: server_like

smb_shares:
  - name: "backup_nas"
    mount_point: "/mnt/backup"
    share_path: "//192.168.1.100/Backup"
    username: "backup_user"
    password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      36393661...
    uid: "root"
    gid: "root"
    dir_mode: "0755"
    file_mode: "0644"
    # allowed_users is omitted here. Access is read-only for everyone based on modes.

  - name: "media_share"
    domain: "MYDOMAIN"
    mount_point: "/mnt/media"
    share_path: "//192.168.1.100/Media"
    username: "media_user"
    password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      66393661...
    uid: "root"
    gid: "media_share_users"
    dir_mode: "0770"
    file_mode: "0660"
    allowed_users:
      - alice
      - bob
```
### For Workstation

```yaml
smb_mount_mode: workstation

smb_shares:
  - name: "backup_nas"
    mount_point: "/mnt/backup"
    share_path: "//192.168.1.100/Backup"
    username: "backup_user"
    password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      36393661...
    uid: "root"
    gid: "root"
    dir_mode: "0755"
    file_mode: "0644"
    # allowed_users is omitted here. Access is read-only for everyone based on modes.

  - name: "media_share"
    domain: "MYDOMAIN"
    mount_point: "/mnt/media"
    share_path: "//192.168.1.100/Media"
    username: "media_user"
    password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      66393661...
    uid: "root"
    gid: "media_share_users"
    dir_mode: "0770"
    file_mode: "0660"
    allowed_users:
      - alice
      - bob
```