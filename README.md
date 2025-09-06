# Ansible Server Preparation Role

## Description
Ansible role for automated server preparation with:
- Disk encryption (LUKS)
- CPU optimization (disable C-states, performance governor)
- Network interface renaming
- Detailed output about CPU, disk encryption and network interface parameters

---
## Supported OS

### ✅ Fully Tested:
- Ubuntu 18.04, 20.04, 22.04
- Debian 11, 12

### 🔄 Limited Support:
- CentOS 7, 8
- RHEL 7, 8
- AlmaLinux, RockyLinux

> *Role is primarily developed for Debian systems. Contributions for better RedHat support are welcome!*

---
## Requirements
- Ansible 2.9+
- Ubuntu 18.04+
- Python 3 on target servers
- `community.crypto` Ansible collection

---
## Prerequisites: Community Module `community.crypto` Installation
```bash
ansible-galaxy collection install community.crypto
```

---
## Variables
| Variable                 | Default             | Description                                                                  |
|--------------------------|---------------------|------------------------------------------------------------------------------|
| `encryption_disk`        | `/dev/xvdf`         | Disk partition to encrypt                                                    |
| `encryption_method`      | `keyfile`           | Encryption method: keyfile or passphrase                                     |
| `encryption_keyfile`     | `/root/luks.key`    | Path to encryption keyfile                                                   |
| `target_interface_name`  | `net0`              | New network interface name                                                   |
| `target_cstate`          | `1`                 | Maximum allowed CPU C-state (e.g., 1 disables deep C-states)                 | 

> **Note:** Variables can be set in `defaults/main.yaml` or passed via `-e`.

---

## Encryption Modes
- keyfile: Automated encryption using keyfile (recommended for servers)
- passphrase: Manual passphrase input required on each boot

---
## Usage

### Configure inventory
The inventory file should contain your server and variables, for example:

```ini
[servers]
3.75.40.188 ansible_user=ubuntu

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
```
### Run playbook
```bash
ansible-playbook -i inventory playbook.yml
```
---
## Features

### Disk Encryption
- LUKS encryption with primary and backup authentication
- Support for both keyfile and passphrase methods
- Safety checks (disk exists, not mounted, not already encrypted)

### CPU Optimization
- Disable CPU C-states
- Set CPU governor to performance mode
- Persistent GRUB settings

> Note1: CPU C-state configuration may not work on virtual machines, depending on hypervisor support.
> Note2: CPU Performace mode control might be limited on virtual machines, depending on hypervisor support

### Network
- Automatic detection of active network interface
- Persistent renaming via udev rules
- Netplan configuration updates

---
## Optional Enhancements
The role includes several features beyond the original task requirements:

### Support both passphrase/key encryption methods 
Role supports passphrase and key as encryption methods 

### Backup passphrase for recovery
A secondary LUKS key/passphrase is created, providing a recovery option in case the primary keyfile or passphrase is lost.

> These enhancements improve convenience and resilience, while maintaining security best practices.

---
## Security
- Keyfile: 64 random characters, permissions `0600` (for "keyfile" mode)
- Backup passphrase provided
- Secure key directory: `/etc/luks-keys/`  

---
## Idempotency
- Tasks check the current system state and only make changes if necessary  
- All tasks are designed to be fully idempotent using appropriate Ansible modules

## Validation
After execution, the playbook will display system information automatically, including:

- Encryption status and method
- CPU governor and C-state settings
- Network interface configuration
- System architecture information

---
## Maintenance: Resetting Encrypted Disk (Optional)
If you need to completely wipe and reinitialize the encrypted disk:

```bash
# Close LUKS mapping (if still open)
sudo cryptsetup luksClose encrypted_data

# Overwrite disk with zeros (irreversible!)
sudo dd if=/dev/zero of=/dev/xvdf bs=1M status=progress
⚠️ Warning: This will irreversibly destroy all data on the disk.
```

---
## Future Improvements
- Extend support to RedHat-based systems (currently untested due to environment availability)
- Encrypted disk automount

---
## Contributing
This role currently has best support for Debian/Ubuntu. Areas for improvement:

- Better RedHat/CentOS support
- Additional network managers support
- Testing on various hypervisors

