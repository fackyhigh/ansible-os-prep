LUKS Encrypt
============

Encrypts disk partitions on a target host using LUKS via `cryptsetup`. The role
locates the root disk, identifies a secondary disk (or neighbouring unmounted
root-disk partitions), optionally creates a GPT partition on the second disk,
initializes LUKS containers, formats them with ext4, and mounts the opened volumes.

Requirements
------------

- Target hosts are Ubuntu/Debian (apt-based).
- Root access (`become: true`).
- Collections: `community.crypto`, `community.general`.
- A second physical disk (with no root partition) is required when
  `create_second_disk_partition: true`.
- Secret provider requirements:
  - `ansible_vault`: `vault_luks_passphrase` must be provided.
  - `hashi_vault`: a running HashiCorp Vault and the `community.hashi_vault`
    collection.
  - `env`: the `LUKS_ENV_PASSPHRASE` environment variable.
  - `plaintext`: passphrase from `luks_passphrase_plaintext` (insecure).

Role Variables
--------------

All variables and their defaults are in `roles/luks_encrypt/defaults/main.yml`:

| Variable                       | Required | Default                    | Choices                        | Comments                                                        |
|--------------------------------|----------|----------------------------|--------------------------------|-----------------------------------------------------------------|
| luks_secret_provider           | yes      | plaintext                  | plaintext, ansible_vault, hashi_vault, env | How the passphrase is resolved.          |
| luks_passphrase_plaintext      | no       | SuperSecureDefaultPassword123! |                          | Passphrase for the `plaintext` provider (insecure).            |
| vault_luks_passphrase          | no       | (empty)                    |                                | Passphrase for the `ansible_vault` provider.                    |
| second_disk_partition_name     | no       | my_partition               |                                | Name of the GPT partition created on the second disk.          |
| create_second_disk_partition   | no       | true                       | true, false                    | Auto-create a GPT partition on the second disk.                |
| partition_filter_min_bytes     | no       | 104857600                  |                                | Min size (bytes) for root-disk neighbour partitions to encrypt.|
| luks_env_passphrase_env_var    | no       | LUKS_ENV_PASSPHRASE        |                                | Env var used by the `env` provider.                            |
| hashi_vault_url                | no       | http://127.0.0.1:8200      |                                | Vault URL used by the `hashi_vault` provider.                  |
| hashi_vault_secret_path        | no       | secret/data/storage/luks   |                                | Vault KVv2 secret path.                                        |
| hashi_vault_secret_field       | no       | password                   |                                | Field holding the passphrase in the Vault secret.              |
| hashi_vault_token_env_var      | no       | VAULT_TOKEN                |                                | Env var holding the Vault token.                                |

Dependencies
------------

No Galaxy-hosted roles are used. Collections `community.crypto` and
`community.general` are required. The `hashi_vault` provider additionally needs the
`community.hashi_vault` collection and a running Vault.

Example Playbook
----------------

    - hosts: all
      become: true
      roles:
        - role: luks_encrypt
          vars:
            luks_secret_provider: plaintext
            create_second_disk_partition: true

Limitations
-----------

- Auto-partitioning of the second disk only happens when the disk is empty; an
  existing non-empty layout halts partition creation.
- The default `plaintext` provider stores the passphrase in clear text and is not
  recommended for production.
- Only a single new partition is created on the second disk; the disk must have no
  root partition.
- The `partition_filter_min_bytes` parameter is used to filter out small partitions that do not require encryption,
  including critical system partitions like the BIOS Boot Partition.