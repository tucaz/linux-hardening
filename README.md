# About this repository

Series of scripts, documents and steps to harden linux installations.

All things described here should apply to most linux distributions, but are tested and based on Ubuntu 18.04.

# SSH

Reload SSH configuration without disconnecting and connecting again to prevent losing access in case you do something wrong [R1]

```sh
systemctl reload ssh.service
```

Backup SSH configuration before changing it [R1]

```sh
cp /etc/ssh/sshd_config /root/sshd_config
```

[sshd_config](https://github.com/tucaz/linux-hardening/blob/master/sshd_config) file with restrictions in place [R1]

- PermitRootLogin no
- MaxAuthTries 3
- PubkeyAuthentication yes
- IgnoreRhosts yes
- PasswordAuthentication no
- PermitEmptyPasswords no
- X11Forwarding no

# References

[R1] - https://linux-audit.com/audit-and-harden-your-ssh-configuration/
