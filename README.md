# ansible-role-borg

Ansible role do install and setup regular backups with [borg](https://github.com/borgbackup/borg). The role does the following

- [Optional] Delete existing repository
- [Initialize a repository](https://borgbackup.readthedocs.io/en/stable/usage/init.html) @ `backup_server`:`target_dir`

   > **Note** In case the repo `target_dir` already exists, the initalization will be skipped

- Create a `systemd` service which regularly (accoring to `backup_schedule`) runs script `borg.sh`

   > **TODO**: add source for `borg.sh` - script is not mine

## Requirements

None

## Role Variables

These are all variables

|Parameter|Description|Default Value|
|---------|-----------|-------------|
|`backup_server`|Name of the server|`SERVER`|
|`backup_user`|Name of the user to connect to the server|`USER`|
|`backup_name`|Name of backup|`test`|
|`backup_encryption_key`|Passphrase for the encryption key using `repokey`|-|
|`backup_port`|Port to connect to `backup_server`|`23`|
|`backup_encryption_method`|Borg [encryption method](https://borgbackup.readthedocs.io/en/stable/usage/init.html#encryption-modes), currently only `repokey` implemented|`repokey`|
|`target_dir`|Target directory of the backups on the `backup_server`|`"./backups/{{ backup_name }}"`|
|`backup_delete`|**WARNING** If set to `true` then existing backup repository will be [deleted](https://borgbackup.readthedocs.io/en/stable/usage/delete.html)|`false`|
|`backup_create`|Creation of repository. You can use the role to explicitly delete an existing `repository` by running the role with `-e backup_delete=true -e backup_create=false`|`true`|
|`backup_schedule`|Systemd schedule notation for the daily backup to run|`*-*-* 03:00:00`|
|`systemd_target_dir`|Location where to copy `.service`-files|`/etc/systemd/system/`|
|`backup_script_dir`|Location where to copy backup script|`/bin/usr/`|
|`backup_source_dir`|Source directory to backup|-|
|`backup_exclude_file`|[`EXCLUDEFILE`](https://borgbackup.readthedocs.io/en/stable/usage/create.html) which contains exclude patterns<br>Takes precedence over `backup_exclude_list`|-|
|`backup_exclude_list`|List of patterns which will be added as `--exclude 'PATTERN'`|-|

To keep sensitive information hidden I recommend to use [`ansible-vault`](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

TODO: Add example

## Dependencies

None

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: localhost
  vars:
  - backup_server: borg.intra
  - backup_user: borguser
  - backup_name: test
  - backup_encryption_key: test
  - backup_port: 23
  - backup_encryption_method: repokey
  - target_dir: "/var/..backups/"
  - backup_exclude_list:
    - "*/Downloads"
    - "*/google-chrome*"
  - backup_source_dir: /home/papanito
  
  roles:
  - role: papanito.borg
```

## License

This is Free Software, released under the terms of the Apache v2 license.

## Author Information

Written by [Papanito](https://wyssmann.com) - [Gitlab](https://gitlab.com/papanito) / [Github](https://github.com/papanito)
