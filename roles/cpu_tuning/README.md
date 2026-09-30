CPU Tuning
==========

Tunes CPU performance on Ubuntu/Debian hosts. It enforces a productive CPU scaling
governor (`performance`) to switch the CPU out of power-saving mode, and disables
deeper idle C-states by injecting `intel_idle.max_cstate` and `processor.max_cstate`
kernel parameters into the GRUB command line, rebuilding the boot configuration and
optionally rebooting the host.

Requirements
------------

- Target hosts are Ubuntu/Debian (apt-based) and use `update-grub`.
- Root access (`become: true`).
- The GRUB default config file must contain a `GRUB_CMDLINE_LINUX_DEFAULT=` line
  with quotes that the role can modify.
- No extra collections are required beyond `ansible.builtin`.

Role Variables
--------------

All variables and their defaults are in `roles/cpu_tuning/defaults/main.yml`:

| Variable                  | Required | Default        | Choices                    | Comments                                                     |
|---------------------------|----------|----------------|----------------------------|--------------------------------------------------------------|
| cpu_scaling_governor      | no       | performance    | performance, powersave, ondemand, etc. | Scaling governor enforced for all cores (productive mode). |
| cpu_cstate_kernel_params  | no       | [intel_idle.max_cstate=1, processor.max_cstate=1] | list of kernel params | C-state limiting parameters injected into GRUB cmdline.     |
| cpu_grub_cfg_path         | no       | /etc/default/grub |                           | GRUB defaults file that is edited.                          |
| cpu_grub_generated_cfg    | no       | /boot/grub/grub.cfg |                      | Generated config verified after `update-grub`.              |
| grub_cstate_required      | no       | x86_64 and true | true, false                | Gates whether the C-state GRUB block runs.                  |
| cpu_tuning_reboot         | no       | true           | true, false                | Reboot the host after applying C-state parameters.           |

Dependencies
------------

No Galaxy-hosted roles are used. The role is self-contained.

Example Playbook
----------------

    - hosts: all
      become: true
      roles:
        - role: cpu_tuning
          vars:
            cpu_scaling_governor: performance
            cpu_tuning_reboot: true

Limitations
-----------

- Supported only on Debian/Ubuntu (uses `cpufrequtils` and `update-grub`).
- On EC2 instance types that do not expose C-state control to the operating system
  (for example `t2.micro`, `t3.*`), the `max_cstate` kernel parameters are written
  to the GRUB configuration but have no real effect after reboot.
- On EC2 the scaling governor is frequently already `performance` by default, so the
  "productive mode" change may be a no-op rather than an actual switch.
- Setting `grub_cstate_required: false` disables the whole C-state block entirely.