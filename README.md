[![CI](https://github.com/guidugli/ansible-role-kernel_config/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-kernel_config/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/release/guidugli/ansible-role-kernel_config?sort=semver)](https://github.com/guidugli/ansible-role-kernel_config/releases)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.kernel__config-blue)](https://galaxy.semaphoreui.com/views/guidugli/ansible-role-kernel_config/overview)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-kernel_config)](https://github.com/guidugli/ansible-role-kernel_config/blob/main/LICENSE)

# Ansible Role: kernel_config

Configure Linux kernel modules, sysctl parameters, and selected udev power-management rules.
This role is aligned to the shared Molecule layout used by the template repository and keeps
role-enforced privilege out of tasks and handlers.

## Requirements

- Supported by the repository metadata template for Fedora, Ubuntu, and Debian platform matrices.
- Requires the collections declared in `requirements.yml`.
- Intended for Linux hosts; some sysctl and service actions are constrained in containers.

## Variables

Variables with defaults from `defaults/main.yml`:

```yaml
kernel_disable_modules:
  - cramfs
  - freevxfs
  - jjfs2
  - hfs
  - hfsplus
  - udf
  - squashfs
  - dccp
  - sctp
  - rds
  - tipc

kernel_blacklist_modules:
  - cramfs
  - freevxfs
  - jjfs2
  - hfs
  - hfsplus
  - udf
  - squashfs
  - dccp
  - sctp
  - rds
  - tipc

kernel_autostart_modules: []

kernel_sysctl:
  - name: fs.suid_dumpable
    value: '0'
  - name: fs.protected_hardlinks
    value: '1'
  - name: fs.protected_symlinks
    value: '1'
  - name: fs.inotify.max_user_instances
    value: '1024'
  - name: kernel.dmesg_restrict
    value: '1'
  - name: kernel.yama.ptrace_scope
    value: '1'
  - name: kernel.randomize_va_space
    value: '2'
  - name: kernel.kptr_restrict
    value: '1'
  - name: kernel.nmi_watchdog
    value: '0'
  - name: net.ipv4.ip_forward
    value: '0'
  - name: net.ipv4.conf.all.forwarding
    value: '0'
  - name: net.ipv4.conf.all.send_redirects
    value: '0'
  - name: net.ipv4.conf.default.send_redirects
    value: '0'
  - name: net.ipv4.conf.all.accept_source_route
    value: '0'
  - name: net.ipv4.conf.default.accept_source_route
    value: '0'
  - name: net.ipv4.conf.all.accept_redirects
    value: '0'
  - name: net.ipv4.conf.default.accept_redirects
    value: '0'
  - name: net.ipv4.conf.all.secure_redirects
    value: '0'
  - name: net.ipv4.conf.default.secure_redirects
    value: '0'
  - name: net.ipv4.conf.all.log_martians
    value: '1'
  - name: net.ipv4.conf.default.log_martians
    value: '1'
  - name: net.ipv4.icmp_echo_ignore_broadcasts
    value: '1'
  - name: net.ipv4.icmp_ignore_bogus_error_responses
    value: '1'
  - name: net.ipv4.conf.all.rp_filter
    value: '1'
  - name: net.ipv4.conf.default.rp_filter
    value: '1'
  - name: net.ipv4.tcp_syncookies
    value: '1'
  - name: net.ipv6.conf.all.disable_ipv6
    value: '1'
  - name: net.ipv6.conf.default.disable_ipv6
    value: '1'

kernel_sysctl_flush_network_routes: true
```

Additional optional udev variables are validated when defined in inventory or playbook vars:
`kernel_udev_sata_link_power_mgmt`, `kernel_udev_disable_bluetooth`,
`kernel_udev_disable_wake_on_lan`, `kernel_udev_usb_autosuspend_devices`,
`kernel_udev_pci_autosuspend_devices`, `kernel_udev_enable_wlan_powersave`,
`kernel_udev_autosuspend_ahci_devices`, and `kernel_udev_autosuspend_scsi_devices`.

## Example playbook

```yaml
---
- name: Configure kernel settings
  hosts: all
  become: true
  roles:
    - role: guidugli.kernel_config
      vars:
        kernel_disable_modules:
          - usb-storage
        kernel_blacklist_modules:
          - firewire-core
        kernel_autostart_modules:
          - br_netfilter
        kernel_sysctl:
          - name: net.ipv4.ip_forward
            value: '0'
          - name: net.ipv4.tcp_syncookies
            value: '1'
```

## Molecule testing

- `molecule/default` and `molecule/systemd` remain generator-controlled.
- Shared converge and verify logic live in `molecule/shared/`.
- Run `molecule test -s default` for the fast container scenario.
- Run `molecule test -s systemd` when you need to exercise the systemd-capable scenario.

## Execution notes

- **Privilege model:** the role never sets `become`; use `become: true` from the calling playbook for real hosts that require privileged writes under `/etc`, package installation, sysctl changes, and service reloads.
- **Containers:** handlers already skip module reloads, route flushes, and udev reloads inside known container runtimes. Some sysctl names can still be restricted by the container runtime or kernel.
- **Systemd:** the module reload handler restarts `systemd-modules-load.service` only when `ansible_facts['service_mgr'] == 'systemd'`.
- **Behavioral caveat:** udev rule template behavior remains unchanged from the source role because only allowed modernization files were edited.
