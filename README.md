Ansible Role: kernel_config
=========

An Ansible Role that install and configure sysctl on RHEL/CentOS, Fedora and Debian/Ubuntu. This role also can disable, blacklist or set kernel modules to autostart. Also, can set UDEV rules related to power management. 

**NOTE:** Disbling kernel modules or settings devices to sleep (udev settings) may impact how the system works and may require system recovery (single user login or even live boot).

Requirements
------------

Operating system running on bare metal or on a hypervisor virtualization. Depending on the settings, this role may not work properly on conteinerized systems.

Role Variables
--------------

**Available variables are listed below, along with default values (see `defaults/main.yml`):**

    #kernel_disable_modules: ['cramfs', 'freevxfs', 'jjfs2', 'hfs', 'hfsplus', 'udf', 'vfat', 'squashfs']

Use the variable below to list the kernel modules that should be disabled you might add usb-storage to the list but this will prevent any usb storage device from working. Consider using USBGuard.

    #kernel_blacklist_modules: ['radeon', 'amdgpu']

Modules to be blacklisted.

    #kernel_autostart_modules: ['br_netfilter']

Modules to be autostarted.

    #kernel_sysctl:
    #  - { name: fs.suid_dumpable, value: "0" }

Specify sysctl parameters to be configured on the system.

    kernel_sysctl_flush_network_routes: yes

If set to yes, it will set net.ipv4.route.flush and net.ipv6.route.flush settings to 1 and reload.

    #kernel_udev_sata_link_power_mgmt: med_power_with_dipm

SATA Link power management policy. Valid values are min_power, max_performance, medium_power, med_power_with_dipm.

    #kernel_udev_autosuspend_ahci_devices: yes

Autosuspend AHCI controllers and ATA devices.

    #kernel_udev_autosuspend_scsi_devices: yes

Autosuspend scsi devices, driver=sd.

    #kernel_udev_disable_bluetooth: no

Disable Bluetooth.

    #kernel_udev_disable_wake_on_lan: yes

Disable wake on lan.

    #kernel_udev_usb_autosuspend_devices:
    #  - { vendor: '13fe', product: '5500' }   # Silicon power flash drive
    #  - { vendor: '1532', product: '0f13', autosuspend: 120 }    # Keyboard

List of devices to autosuspend.
Usually linux already set autosuspend for usb hubs and other devices, so do not need to use this option for those. Check which devices are set to auto or not looking at /sys/bus/usb/devices/*/power/control.

    #kernel_udev_pci_autosuspend_devices:
    #  - { vendor: '0x8086', device: '0x0c00' }
    #  - { vendor: '0x8086', device: '0x0412' }

List of pci devices to autosuspend.

    #kernel_udev_enable_wlan_powersave: yes

Enable wlan power save?

**The variables listed below do not need to be changed for targeted systems (see `vars/main.yml`):**

    kernel_udev_reload_cmd: "udevadm control --reload-rules && udevadm trigger"

Command to reload udev rules.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      vars:
        kernel_disable_modules: ['cramfs', 'freevxfs', 'jjfs2', 'hfs', 'hfsplus', 'udf', 'vfat', 'squashfs']
        kernel_blacklist_modules: ['radeon', 'amdgpu']
        kernel_sysctl:
          - { name: net.ipv4.conf.all.forwarding, value: "0" }
          - { name: net.ipv4.conf.all.send_redirects, value: "0" }
          - { name: net.ipv4.conf.default.send_redirects, value: "0" }
          - { name: net.ipv4.conf.all.accept_source_route, value: "0" }
          - { name: net.ipv4.conf.default.accept_source_route, value: "0" }
          - { name: net.ipv4.conf.all.accept_redirects, value: "0" }
          - { name: net.ipv4.conf.default.accept_redirects, value: "0" }
          - { name: net.ipv4.conf.all.secure_redirects, value: "0" }
          - { name: net.ipv4.conf.default.secure_redirects, value: "0" }
          - { name: net.ipv4.conf.all.log_martians, value: "1" }
          - { name: net.ipv4.conf.default.log_martians, value: "1" }
          - { name: net.ipv4.icmp_echo_ignore_broadcasts, value: "1" }
          - { name: net.ipv4.icmp_ignore_bogus_error_responses, value: "1" }
          - { name: net.ipv4.conf.all.rp_filter, value: "1" }
          - { name: net.ipv4.conf.default.rp_filter, value: "1" }
          - { name: net.ipv4.tcp_syncookies, value: "1" }
        kernel_sysctl_flush_network_routes: yes

      roles:
         - { role: guidugli.kernel_config }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
