Ansibible OS Prep Playbook
==========================

Prepares target Linux servers by running the `luks_encrypt`, `network_tuning`, and
`cpu_tuning` roles in this order: disk encryption, network interface rename, and
CPU performance tuning. After all roles complete, a final block collects and prints
CPU topology information.

Requirements
------------

- Ansible Core 2.14 or newer.
- Target hosts are Ubuntu/Debian (apt-based) with `become` (root) access.
- Collections used by the roles:
  - `community.crypto` (LUKS)
  - `community.general` (parted, filesystem, mount)
  - `community.hashi_vault` (optional, only for the `hashi_vault` LUKS secret provider)
- A running HashiCorp Vault is required only if `luks_secret_provider: hashi_vault`.
- Inventory group `target_servers` must be defined.

Role Variables
--------------

The playbook itself does not define role variables; it passes none to the roles.
The CPU summary block at the end uses the derived fact `cpu_vendor_name`:

| Variable       | Required | Default | Choices | Comments                                        |
|----------------|----------|---------|---------|-------------------------------------------------|
| cpu_vendor_name| no       | Derived | Intel, AMD, Unknown | Computed from the CPU vendor string in facts |

See each role's README for all input variables.

Example Playbook
----------------

    - name: Prepare OS Configuration (LUKS, CPU, Network)
      hosts: target_servers
      become: true

      roles:
        - role: luks_encrypt
        - role: network_tuning
        - role: cpu_tuning

Limitations
-----------

- Only Ubuntu/Debian targets with `update-grub` and netplan are supported.
- On EC2 burstable instance types that do not expose C-state control to the OS
  (for example `t2.micro`, `t3.*`), the CPU C-state kernel parameters are written
  to the GRUB configuration but have no actual effect.
- The `network_tuning` role applies the interface rename only after a system reboot.