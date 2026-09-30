Network Tuning
==============

Renames the active network interface within the netplan configuration on
Ubuntu/Debian hosts. The role discovers the active interface that carries an IPv4
address, updates the `set-name:` directive in the found netplan YAML file, notifies a
reboot to apply the rename, and verifies the interface exists with the target name
after reboot.

Requirements
------------

- Target hosts are Ubuntu/Debian using netplan (`/etc/netplan/*.yaml` or `*.yml`).
- Root access (`become: true`).
- No extra collections are required beyond `ansible.builtin`.

Role Variables
--------------

All variables and their defaults are in `roles/network_tuning/defaults/main.yml`:

| Variable                  | Required | Default | Choices   | Comments                                             |
|---------------------------|----------|---------|-----------|------------------------------------------------------|
| network_interface_rename  | no       | net0    |           | Target interface name after renaming.                |
| network_reboot_after_rename | no     | true    | true, false | Reboot the host to apply the netplan rename.       |

Dependencies
------------

No Galaxy-hosted roles are used. The role is self-contained.

Example Playbook
----------------

    - hosts: all
      become: true
      roles:
        - role: network_tuning
          vars:
            network_interface_rename: net0
            network_reboot_after_rename: true

Limitations
-----------

- Only netplan-based configurations under `/etc/netplan` are supported.
- The interface rename takes effect only after a system reboot.
- If `network_reboot_after_rename: true` the host is rebooted by the role's handler.
- If no netplan configuration file is found, or no active IPv4 interface can be
  auto-discovered, the role fails.