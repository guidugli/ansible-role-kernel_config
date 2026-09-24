[![CI](https://github.com/guidugli/ansible-role-kernel_config/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-kernel_config/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-kernel_config?sort=semver&label=release)](https://github.com/guidugli/ansible-role-kernel_config/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.kernel__config-blue)](https://galaxy.semaphoreui.com/views/guidugli/ansible-role-kernel_config/overview)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-kernel_config)](https://github.com/guidugli/ansible-role-kernel_config/blob/main/LICENSE)

# Ansible Role: kernel_config

Configures Linux kernel module policy, persistent sysctl settings, and selected udev power-management rules. The role follows the shared repository template standards: generator-safe metadata, shared Molecule coverage, deterministic task behavior, external privilege control, container-aware runtime operations, and systemd-guarded handlers.

## Requirements

- Ansible Core 2.14 or newer, as declared by role metadata.
- Linux targets using an apt-family or RPM-family package manager.
- Root-level permissions on real hosts for package installation and writes under `/etc`; provide privilege escalation from the calling playbook, inventory, or automation controller.
- `containers.podman` collection version 1.10.0 or newer for Molecule container scenarios.
- The role does not require `ansible.posix` at runtime; sysctl persistence and runtime application use Ansible built-in modules plus the system `sysctl` command.

## Features

- Writes deterministic `/etc/modprobe.d/<module>-disabled.conf` files to prevent selected modules from loading through `install <module> /bin/true` rules.
- Writes deterministic `/etc/modprobe.d/<module>-blacklist.conf` files to blacklist selected kernel modules.
- Writes `/etc/modules-load.d/<module>.conf` files for modules that should load at startup.
- Installs distribution-appropriate procps and udev package dependencies using apt or RPM-family package managers.
- Repairs missing apt package indexes in minimal Debian/Ubuntu containers before package installation, then uses a stale-cache guard for normal apt installs.
- Flushes pending module reload handlers before sysctl work so module-load policy is applied before kernel parameter enforcement.
- Persists sysctl values in `/etc/sysctl.d/99-kernel_config.conf` using one managed line per requested key.
- Applies changed runtime sysctl values on non-container hosts only.
- Flushes IPv4 and IPv6 route caches through a handler when runtime sysctl values change and route flushing is enabled.
- Ensures `/etc/udev/rules.d` exists before rendering udev rules.
- Renders udev rules for SATA link power management, Bluetooth rfkill policy, USB autosuspend and wakeup, Wi-Fi power saving, Wake-on-LAN disablement, PCI autosuspend, AHCI/ATA autosuspend, and SCSI autosuspend.
- Skips module reload, runtime sysctl application, network route flush, and udev reload/trigger operations inside known container runtimes.
- Restarts `systemd-modules-load.service` only on systemd-managed, non-container hosts.
- Provides shared Molecule converge and verify logic for both default and systemd scenarios.

## Supported platforms

Role metadata declares support for Fedora, Ubuntu, and Debian. The shared Molecule matrix currently covers Fedora 44 and 43, Ubuntu 26.04 and 24.04, and Debian 13 and 12.

## Variables

All role inputs with defaults are defined in `defaults/main.yml` and validated in `meta/argument_specs.yml`. Optional udev variables are validated when defined by inventory or playbook data.

### Core variables

| Variable | Type | Default | Description |
|---|---|---|---|
| `kernel_disable_modules` | list of strings | `cramfs`, `freevxfs`, `jjfs2`, `hfs`, `hfsplus`, `udf`, `squashfs`, `dccp`, `sctp`, `rds`, `tipc` | Modules for which the role creates `/etc/modprobe.d/<module>-disabled.conf` with `install <module> /bin/true`. Use this when the module should be prevented from loading even if another component requests it. |
| `kernel_blacklist_modules` | list of strings | `cramfs`, `freevxfs`, `jjfs2`, `hfs`, `hfsplus`, `udf`, `squashfs`, `dccp`, `sctp`, `rds`, `tipc` | Modules for which the role creates `/etc/modprobe.d/<module>-blacklist.conf` with `blacklist <module>`. Use this for normal module blacklist policy. |
| `kernel_autostart_modules` | list of strings | `[]` | Modules to load at startup through `/etc/modules-load.d/<module>.conf`. Leave empty when no startup-loaded modules are required. |
| `kernel_sysctl` | list of dictionaries | See default sysctl table below | Persistent sysctl policy. Each item requires `name` and `value`. The role writes these values to `/etc/sysctl.d/99-kernel_config.conf`; on non-container hosts it also applies runtime values if they differ. |
| `kernel_sysctl_filename` | name of the file to contain sysctl policies | 99-kernel_config.conf | Sets the file name to be created under /etc/sysctl.d that will contain the sysctl policies |
| `kernel_sysctl_flush_network_routes` | boolean | `true` | When runtime sysctl values change on non-container hosts, this notifies the route-flush handler for `net.ipv4.route.flush` and `net.ipv6.route.flush`. |

### Default sysctl policy

| Sysctl name | Default value | Purpose |
|---|---:|---|
| `fs.suid_dumpable` | `'0'` | Prevent unrestricted core dumping of setuid executables. |
| `fs.protected_hardlinks` | `'1'` | Enable hardlink-related protections. |
| `fs.protected_symlinks` | `'1'` | Enable symlink-related protections. |
| `fs.inotify.max_user_instances` | `'1024'` | Set the per-user inotify instance limit. |
| `kernel.dmesg_restrict` | `'1'` | Restrict unprivileged access to the dmesg buffer. |
| `kernel.yama.ptrace_scope` | `'1'` | Restrict ptrace attach behavior to predefined relationships. |
| `kernel.randomize_va_space` | `'2'` | Enable full address-space layout randomization. |
| `kernel.kptr_restrict` | `'1'` | Restrict exposure of kernel pointers. |
| `kernel.nmi_watchdog` | `'0'` | Disable NMI hard-lockup watchdog detection by default. |
| `net.ipv4.ip_forward` | `'0'` | Disable IPv4 forwarding. |
| `net.ipv4.conf.all.forwarding` | `'0'` | Disable IPv4 forwarding for all interfaces. |
| `net.ipv4.conf.all.send_redirects` | `'0'` | Disable IPv4 redirect sending on all interfaces. |
| `net.ipv4.conf.default.send_redirects` | `'0'` | Disable IPv4 redirect sending for new interfaces. |
| `net.ipv4.conf.all.accept_source_route` | `'0'` | Disable IPv4 source-routed packets on all interfaces. |
| `net.ipv4.conf.default.accept_source_route` | `'0'` | Disable IPv4 source-routed packets for new interfaces. |
| `net.ipv4.conf.all.accept_redirects` | `'0'` | Disable IPv4 ICMP redirect acceptance on all interfaces. |
| `net.ipv4.conf.default.accept_redirects` | `'0'` | Disable IPv4 ICMP redirect acceptance for new interfaces. |
| `net.ipv4.conf.all.secure_redirects` | `'0'` | Disable secure IPv4 redirects on all interfaces. |
| `net.ipv4.conf.default.secure_redirects` | `'0'` | Disable secure IPv4 redirects for new interfaces. |
| `net.ipv4.conf.all.log_martians` | `'1'` | Log suspicious IPv4 packets on all interfaces. |
| `net.ipv4.conf.default.log_martians` | `'1'` | Log suspicious IPv4 packets on new interfaces. |
| `net.ipv4.icmp_echo_ignore_broadcasts` | `'1'` | Ignore ICMP echo broadcasts. |
| `net.ipv4.icmp_ignore_bogus_error_responses` | `'1'` | Ignore bogus ICMP error responses. |
| `net.ipv4.conf.all.rp_filter` | `'1'` | Enable strict reverse-path filtering on all interfaces. |
| `net.ipv4.conf.default.rp_filter` | `'1'` | Enable strict reverse-path filtering for new interfaces. |
| `net.ipv4.tcp_syncookies` | `'1'` | Enable TCP SYN cookies. |
| `net.ipv6.conf.all.disable_ipv6` | `'1'` | Disable IPv6 on all interfaces. |
| `net.ipv6.conf.default.disable_ipv6` | `'1'` | Disable IPv6 for new interfaces. |

### Sysctl policy notes and suggestions

The default `kernel_sysctl` list is a conservative non-router hardening baseline. It is intentionally opinionated and should be reviewed before applying it to routers, firewalls, Kubernetes nodes, multi-homed hosts, developer workstations, or hosts that require IPv6.

| Sysctl | Default | Suggested review |
|---|---:|---|
| `net.ipv6.conf.all.disable_ipv6`, `net.ipv6.conf.default.disable_ipv6` | `'1'` | The role disables IPv6 by default. Override these values to `'0'` when the host participates in IPv6 networking or when applications, overlay networks, service discovery, compliance rules, or cloud platform defaults expect IPv6 to remain enabled. |
| `net.ipv4.conf.all.rp_filter`, `net.ipv4.conf.default.rp_filter` | `'1'` | Strict reverse-path filtering helps reduce spoofed traffic, but it can break asymmetric routing, policy routing, and multi-homed designs. Use `'2'` for loose mode, or apply interface-specific overrides, when strict reverse-path validation is not compatible with the network design. |
| `kernel.nmi_watchdog` | `'0'` | This disables NMI hard-lockup detection. Keep `'0'` when reducing virtualized-host watchdog noise is preferred. Use `'1'` on physical hosts or troubleshooting targets where hard-lockup detection is operationally required. |
| `kernel.kptr_restrict` | `'1'` | This is a valid restriction level. Environments with stricter kernel pointer exposure requirements may choose `'2'`. |
| `kernel.yama.ptrace_scope` | `'1'` | This restricts ptrace while still preserving common parent-child debugging flows. Consider `'2'` for admin-only ptrace on production servers. Avoid `'3'` unless intentionally disabling ptrace attach until reboot is acceptable. |
| `net.ipv4.conf.all.accept_redirects`, `net.ipv4.conf.default.accept_redirects` | `'0'` | Keep disabled for hardened non-router baselines. Review before applying to networks that intentionally use ICMP redirects for local routing behavior. |

Example override for an IPv6-enabled, multi-homed host:

```yaml
kernel_sysctl:
  - name: net.ipv6.conf.all.disable_ipv6
    value: '0'
  - name: net.ipv6.conf.default.disable_ipv6
    value: '0'
  - name: net.ipv4.conf.all.rp_filter
    value: '2'
  - name: net.ipv4.conf.default.rp_filter
    value: '2'
  - name: kernel.nmi_watchdog
    value: '1'
  - name: kernel.kptr_restrict
    value: '2'
  - name: kernel.yama.ptrace_scope
    value: '2'
```

The role does not include older hardening recommendations such as changing `net.ipv4.tcp_timestamps`, and it does not manage deprecated route-cache sysctls. Add extra sysctls only after checking the target kernel documentation and the host's routing, debugging, and IPv6 requirements.

### Optional udev variables

| Variable | Type | Default | Description |
|---|---|---|---|
| `kernel_udev_sata_link_power_mgmt` | string | undefined | When defined, renders a SATA link power-management rule. Accepted values are `min_power`, `max_performance`, `medium_power`, and `med_power_with_dipm`. |
| `kernel_udev_disable_bluetooth` | boolean | undefined | When `true`, renders a Bluetooth rfkill rule that sets Bluetooth state to disabled. When undefined or false, the Bluetooth rule template renders no active rule. |
| `kernel_udev_disable_wake_on_lan` | boolean | undefined | Validated for compatibility with inventory or playbook data. The current Wake-on-LAN template always renders a rule that disables WoL for `enp*` interfaces, independent of this variable. |
| `kernel_udev_usb_autosuspend_devices` | list of dictionaries | undefined | Renders USB autosuspend and wakeup rules for listed devices. Each entry is expected to provide `vendor` and `product`; optional `autosuspend` sets a positive autosuspend delay. |
| `kernel_udev_pci_autosuspend_devices` | list of dictionaries | undefined | Renders PCI autosuspend rules for listed devices. Each entry is expected to provide `vendor` and `device`. |
| `kernel_udev_enable_wlan_powersave` | boolean | undefined | When `true`, Wi-Fi power saving is enabled for `wl*` interfaces. When undefined or false, the template renders a rule that disables Wi-Fi power saving. |
| `kernel_udev_autosuspend_ahci_devices` | boolean | undefined | When defined, renders AHCI controller and ATA device autosuspend rules. The current template checks whether the variable is defined, not whether it is true. |
| `kernel_udev_autosuspend_scsi_devices` | boolean | undefined | When defined, renders SCSI disk autosuspend rules. The current template checks whether the variable is defined, not whether it is true. |

### Internal variables

| Variable | Scope | Description |
|---|---|---|
| `kernel_procps_packages` | internal, from `vars/main.yml` | Resolves the procps/udev package list for the detected distribution. Default apt-family value is `procps` and `udev`; Red Hat, Rocky, CentOS, Fedora, and Archlinux map to `procps-ng`. |
| `_container_types` | internal, from `vars/main.yml` | Container runtime names used by task and handler guards: `docker`, `podman`, `lxc`, `containerd`, and `container`. |

## Example playbook

```yaml
---
- name: Configure kernel policy
  hosts: linux_hosts
  become: true
  roles:
    - role: guidugli.kernel_config
      vars:
        kernel_disable_modules:
          - usb-storage
          - firewire-core
        kernel_blacklist_modules:
          - cramfs
          - udf
        kernel_autostart_modules:
          - br_netfilter
        kernel_sysctl:
          - name: net.ipv4.ip_forward
            value: '0'
          - name: net.ipv4.tcp_syncookies
            value: '1'
          - name: net.ipv6.conf.all.disable_ipv6
            value: '0'
          - name: net.ipv6.conf.default.disable_ipv6
            value: '0'
        kernel_udev_sata_link_power_mgmt: min_power
        kernel_udev_enable_wlan_powersave: true
        kernel_udev_usb_autosuspend_devices:
          - vendor: idVendor
            product: idProduct
            autosuspend: 2
```

## Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

The default scenario provides fast coverage across minimal Podman containers. The systemd scenario prepares systemd-capable containers and exercises systemd-aware execution paths. Shared converge and verify logic lives under `molecule/shared`.

## Execution notes

- **Privilege model:** The role never declares `become`, `become_user`, or `become_method`. Use `become: true` from the calling playbook, inventory, or automation controller for real hosts because package installation, `/etc` writes, sysctl changes, systemd reloads, and udev commands are privileged operations.
- **Container behavior:** Known Docker, Podman, LXC, containerd, and generic container guests skip runtime sysctl application, module reload, route flush, and udev reload/trigger handlers. Persistent files are still managed and verified in containers.
- **Systemd behavior:** `systemd-modules-load.service` is restarted only when `ansible_facts['service_mgr'] == 'systemd'` and the target is not a known container runtime.
- **APT behavior:** Minimal Debian and Ubuntu containers may start without package index files. The role checks `/var/lib/apt/lists` and refreshes apt metadata only when indexes are missing, then uses `cache_valid_time: 3600` on apt package installation for idempotent cache handling.
- **Sysctl behavior:** The role persists sysctl settings in `/etc/sysctl.d/99-kernel_config.conf`. Runtime `sysctl -w` changes are applied only on non-container hosts to avoid repeated changes and unsupported kernel namespace behavior in containers.
- **Udev behavior:** The role ensures `/etc/udev/rules.d` exists before rendering rules. Reload and trigger operations are handlers and are skipped in known containers.
- **Known behavior retained:** The AHCI and SCSI autosuspend templates render when their matching variables are defined, even when defined as `false`. The Wake-on-LAN template disables WoL independently of `kernel_udev_disable_wake_on_lan`.

## Release workflow

Generated metadata and inventories are refreshed through the repository scripts. Review generated changes before tagging a release.

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

## License

MIT

## Author

Carlos Guidugli
