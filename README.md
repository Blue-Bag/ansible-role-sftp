# SFTP-Server

An Ansible role which configures an OpenSSH server for chrooted SFTP access.  The role is built in such a way that it will not unnecessarily alter a user's OpenSSH customisations.  Instead, it simply changes the crucial bits that it needs to, and adds the rest of its configuration in the form of a custom config block (OpenSSH's lack of some form of conf.d/ support forces this behaviour).

This forked version has cherry picked additions/updates form other forks
These are cited in the various commits where relevant.
They have been manually applied cf. cherry picking as they may have been adapted.
Buy props to the other contributors.

This Forked version also contains the following additional facilities:

 - Logging of the sftp connection activity
 - Rotation of the sftp log
 - configuration of the nologin shell
 - removal of bash user files created from skeleton (ideally these would not be created)

This role can be used in conjunction with the incron role (vkil.incron) to provide notification
or other actions when files are dropped via sftp.

## Requirements

It is advisable that `scp_if_ssh` be set to `true` in the `ssh_connection` section of your `ansible.cfg` file, seeing as how Ansible uses SFTP for file transfers by default, and you can easily lock yourself out of your server's SFTP by using this role.  The SCP fallback will continue to work.  Example config:

```ini
; ansible.cfg
...
[ssh_connection]
scp_if_ssh=True
```

Other than that, only Ansible itself is required.  Tested using Ansible 1.9, 2.0.2.0 and 2.1.0.0.  Works on Ubuntu 14.04 and 16.04, untested on other versions.

## Role Variables

####The following role variables are relevant:

 * `sftp_home_partition`: The partition where SFTP users' home directories will be located.  Defaults to "/home".
 * `sftp_group_name`: The name of the Unix group to which all SFTP users must belong.  Defaults to "sftpusers".
 * `sftp_directories`: A list of directories that need to be created automatically for each SFTP user.  Defaults to a blank list (i.e. "[]").
 * Values can be plain strings, or dictionaries containing `name` and (optionally) `mode` key/value pairs.
 * `sftp_allow_passwords`: Whether or not to allow password authentication for SFTP. Defaults to False.
 * `sftp_users`: A list of users, in map form, containing the following elements:
  * `name`: The Unix name of the user that requires SFTP access.
  * `password`: A password hash for the user to login with.  Blank passwords can be set with `password: ""`.  NOTE: It appears that `UsePAM yes` and `PermitEmptyPassword yes` need to be set in `sshd_config` in order for blank passwords to work properly.  Making those changes currently falls outside the scope of this role and will need to be done externally.
  * `authorized`: A list of files placed in `files/` which contain valid public keys for the SFTP user.

##### Logging SFTP activity

 * `sftp_custom_log`: Boolean to set up custom logging
 * `sftp_log_file`: The name of the custom log file (including full path)
 * `sftp_log_target`:  the default target of the log e.g. 'AUTH'
 * `sftp_log_level`:  the level of logging' e.g. `VERBOSE`, `INFO` etc
 * `sftp_rsyslog_conf`: rsyslog configuration of the custom log

#####Default user files

When a new user is created, bashrc and other profile files may be created, these are generally not required as we
are not allowing interactive shells. Ideally they would not be created. Could be done by using a custom blank skeleton set? But for now we can pass in a set of files to delete from the user's home.

*`sftp_user_files_delete`:  User home directory files to delete

 - e.g. [`.bashrc`, `.profile`, `.bash_logout`]

## Example Playbook

```yaml
---
- name: test-playbook | Test sftp-server role
  hosts: all
  become: yes
  become_user: root
  vars:
    - sftp_users:
      - name: peter
        password: "$1$salty$li5TXAa2G6oxHTDkqx3Dz/" # passpass
        authorized: []
        shell: /sbin/nologin
        directories:
          - { name: inbox, mode: 755}
      - name: sally
        password: ""
        authorized: [sally.pub]

  roles:
    - sftp-server
```

When sftp_force_command is false, we can force a per user specific
command by putting the "command" option in the .ssh/authorized_keys for
the user in question.

Example:
/home/user/.ssh/authorized_keys
command="rsync --server -av --delete . /my/dir", ssh-rsa AAAA....
see https://github.com/rastandy/ansible-sftp/commit/490cf934f84ed407e8c1945e20854fa8f712c391

Logging:
Custom loggin can be set up to enable the logging of sftp activity.
see:
 - http://www.the-art-of-web.com/system/sftp-logging-chroot/
 - https://debian-administration.org/article/637/OpenSSH_logging_with_ChrootDirectory

## License

Licensed under the MIT License. See the LICENSE file for details.
