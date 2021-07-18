# Ansible role "papanito.borg" <!-- omit in toc -->

[![Ansible Role](https://img.shields.io/ansible/role/47022)](https://galaxy.ansible.com/papanito/borg) [![Ansible Quality Score](https://img.shields.io/ansible/quality/47022)](https://galaxy.ansible.com/papanito/borg) [![Ansible Role](https://img.shields.io/ansible/role/d/47022)](https://galaxy.ansible.com/papanito/borg) [![GitHub issues](https://img.shields.io/github/issues/papanito/ansible-role-borg)](https://github.com/papanito/ansible-role-borg/issues) [![GitHub pull requests](https://img.shields.io/github/issues-pr/papanito/ansible-role-borg)](https://github.com/papanito/ansible-role-borg/pulls)

Ansible role do install and setup regular backups with [borg](https://github.com/borgbackup/borg). The role does the following

- [Optional] Delete existing repository
- [Initialize a repository](https://borgbackup.readthedocs.io/en/stable/usage/init.html) at `protocol`://`backup_server`:`target_dir` or `target_dir`

   > **Notes**
   >
   > In case the repo `target_dir` already exists, the initalization will be skipped
   > If `backup_server` is not specified role assumes a local backup i.e. to a local directory

- Create a `systemd` service which regularly (accoring to `backup_schedule`) runs script `borg.sh` from [borgbackup.org](https://borgbackup.readthedocs.io/en/stable/quickstart.html#automating-backups)
- There will be an individual borg-script named `automatic-backup-{{service_name}}.sh` in `/opt/borg_backup` which is customized with

  - `backup_source_dir`
  - `backup_exclude_file` or `backup_exclude_list`
  - `backup_schedule`

## Requirements

None

## Role Variables

These are all variables

| Parameter                  | Description                                                                                                                                                        | Default Value                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------- |
| `backup_server`            | Name of the backup server - if not defined, it assumes a local backup                                                                                              | -                               |
| `backup_user`              | Name of the user to connect to the server                                                                                                                          | -                               |
| `backup_port`              | Port to connect to `backup_server`                                                                                                                                 | -                               |
| `protocol`                 | Protocol used to connect to `backup_server`                                                                                                                        | `ssh`                           |
| `backup_name`              | [mandatory] Name of backup                                                                                                                                         |                                 |
| `backup_encryption_key`    | [mandatory] Passphrase for the encryption key using `repokey`                                                                                                      | -                               |
| `backup_encryption_method` | Borg [encryption method](https://borgbackup.readthedocs.io/en/stable/usage/init.html#encryption-modes), currently only `repokey` implemented                       | `repokey`                       |
| `target_dir`               | Target directory of the backups on the `backup_server`                                                                                                             | `"./backups/{{ backup_name }}"` |
| `backup_delete`            | **WARNING** If set to `true` then existing backup repository will be [deleted](https://borgbackup.readthedocs.io/en/stable/usage/delete.html)                      | `false`                         |
| `backup_create`            | Creation of repository. You can use the role to explicitly delete an existing `repository` by running the role with `-e backup_delete=true -e backup_create=false` | `true`                          |
| `backup_schedule`          | Systemd schedule notation for the daily backup to run                                                                                                              | `*-*-* 03:00:00`                |
| `backup_include_list`      | List of source directories to backup                                                                                                                               | -                               |
| `backup_exclude_file`      | [`EXCLUDEFILE`](https://borgbackup.readthedocs.io/en/stable/usage/create.html) which contains exclude patterns<br>Takes precedence over `backup_exclude_list`      | -                               |
| `backup_exclude_list`      | List of patterns which will be added as `--exclude 'PATTERN'`                                                                                                      | -                               |

The following parameters are related to the systemd service file:

| Parameter            | Description                                                  | Default Value          |
| -------------------- | ------------------------------------------------------------ | ---------------------- |
| `systemd_target_dir` | Location where to copy `.service`-files                      | `/etc/systemd/system/` |
| `borg_systemd_user`       | User for systemd service                                     | `backup`               |
| `borg_systemd_group`      | Group for systemd service                                    | `backup`               |
| `borg_systemd_on_failure` | If set it will make an [OnFailure] entry in the service file | `-`                    |
| `systemd_script_mode`        | Mode of the script file                                  | `0774`                        |
| `systemd_service_mode`        | Mode of the service file                                  | `0644`                        |

The script which is deployed also defines the options for `prune` as described at [borg prune](https://borgbackup.readthedocs.io/en/stable/usage/prune.x
html). Values which expect a number but variable is not defined, then the option is not provided.

| Parameter                    | Description                                                          | Default Value |
| ---------------------------- | -------------------------------------------------------------------- | ------------- |
| `backup_prune_dryrun`        | `-n, --dry-run` do not change repository                             | `false`       |
| `backup_prune_force`         | `--force` force pruning of corrupted archives                        | `false`       |
| `backup_prune_stats`         | `-s, --stats` print statistics for the deleted archive               | `true`        |
| `backup_prune_list`          | `--list` output verbose list of archives it keeps/prunes             | `true`        |
| `backup_prune_keep_within`   | `--keep-within INTERVAL` keep all archives within this time interval | -             |
| `backup_prune_keep_last`     | `--keep-last, --keep-secondly` number of secondly archives to keep   | -             |
| `backup_prune_keep_minutely` | `--keep-minutely` number of minutely archives to keep                | -             |
| `backup_prune_keep_hourly`   | `-H, --keep-hourly` number of hourly archives to keep                | -             |
| `backup_prune_keep_daily`    | `-d, --keep-daily` number of daily archives to keep                  | -             |
| `backup_prune_keep_weekly`   | `-w, --keep-weekly` number of weekly archives to keep                | -             |
| `backup_prune_keep_monthly`  | `-m, --keep-monthly` number of monthly archives to keep              | -             |
| `backup_prune_keep_yearly`   | `-y, --keep-yearly` number of yearly archives to keep                | -             |
| `backup_prune_save_space`    | `--save-space` work slower, but using less space                     | `false`       |

To keep sensitive information hidden I recommend to use [`ansible-vault`](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

You can define the passowrd file in `ansible.cfg` so none vault parameter has to be specified. Thus, the encrypted variable `backup_encryption_key` can be created as follows:

```bash
ansible-vault encrypt_string  'SupersecretPa$$phrase' --name 'backup_encryption_key'
```

## Dependencies

None

## Examples

### Example Playbook remote backup

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: localhost
  vars:
  - backup_server: borg.intra
  - backup_user: borguser
  - backup_name: mybackupname
  - backup_encryption_key: test
  - backup_port: 23
  - target_dir: "/var/backups/"
  - backup_schedule: "*-*-* 03:00:00"
  - backup_exclude_list:
    - "*/Downloads"
    - "*/google-chrome*"
  - backup_include_list:
    - /home/papanito
  - backup_prune_keep_daily: 7
  - backup_prune_keep_weekly: 5
  - backup_prune_keep_monthly: 6
  - backup_prune_keep_yearly: 1
  
  roles:
  - role: papanito.borg
```

This will create a backup at `ssh://borguser@borg.intra:/var/backup/mybackupname` and the following systemd files

- `/opt/borg_backup/automatic-backup-mybackupname-borg.intra.sh` (backup script)
- `/etc/systemd/system/automatic-backup-mybackupname-borg.intra.service` (systemd service file)
- `/etc/systemd/system/automatic-backup-mybackupname-borg.intra.timer` (systemd timers file)

### Example Playbook local backup

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: localhost
  vars:
  - backup_name: mybackupname
  - backup_encryption_key: test
  - target_dir: "/var/backup/"
  - backup_schedule: "*-*-* 03:00:00"
  - backup_exclude_list:
    - "*/Downloads"
    - "*/google-chrome*"
  - backup_include_list:
    - /home/papanito
  - backup_prune_keep_daily: 7
  - backup_prune_keep_weekly: 5
  - backup_prune_keep_monthly: 6
  - backup_prune_keep_yearly: 1
```

This will create a backup at `/var/backup/mybackupname` and the following systemd files

- `/opt/borg_backups/automatic-backup-mybackupname-local.sh` (backup script)
- `/etc/systemd/system/automatic-backup-mybackupname-local.service` (systemd service file)
- `/etc/systemd/system/automatic-backup-mybackupname-local.timer` (systemd timers file)

## License

This is Free Software, released under the terms of the Apache v2 license.

## Author Information

Written by [Papanito](https://wyssmann.com) - [Gitlab](https://gitlab.com/papanito) / [Github](https://github.com/papanito)


[OnFailure]: https://dyn.manpages.debian.org/buster/systemd/systemd.unit.5.en.html#%5BUNIT%5D_SECTION_OPTIONS[OnFailure]