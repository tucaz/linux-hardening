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

# Creating non-root user [R2]

```sh
sudo adduser <username>
sudo usermod -aG sudo <username>
```

Test that the user works by running:

```
su - <username>
```

You will be logged in as <username> and should be able to see it's home directory `/home/<username>` and not access `/root/`

```sh
ls /root
ls: cannot open directory '/root': Permission denied
```
Once your user is created we need to give SSH access to it. The first step is to create a SSH key locally on the client that will be used to authenticate with the server [R3].

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

The `filename.pub` file is the public key and the other `filename` is your private key. Don't lose that file.

*Whenever you are creating a SSH key I recommend you to use a password. I don't, but it's at my own risk. Without a password anyone with your private key can access the server*

Now it's time to copy the public key into the server and add it to the list of authorized keys for your newly created user. I usually use `scp` from my local machine, but copy and paste can also be used [R4].

On the client:

```sh
cd /user/.ssh/
scp filename.pub root@your_server_address:/home/<username>/
```

On the server:

```sh
mkdir /home/<username>/.ssh
chmod 700 /home/<username>/.ssh
mv /home/<username>/filename.pub /home/<username>/.ssh/authorized_keys
chmod 600 /home/<username>/.ssh/authorized_keys
```

Finally, let's try to login as the new user using the new key before we reload SSH and lose SSH access if we did something wrong.

On the client:

```sh
ssh -i filename.pub <username>@your_server_address
```

If you can login, and only if you can successfully login, you should now restart the SSH service in order to disable root access from SSH:

```sg
systemctl reload ssh.service
```

You can also remove the backup file you had:

```sh
rm /root/sshd_config
```

# References

- [R1] - https://linux-audit.com/audit-and-harden-your-ssh-configuration/
- [R2] - https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
- [R3] - https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
- [R4] - https://linux-audit.com/using-ssh-keys-instead-of-passwords/
