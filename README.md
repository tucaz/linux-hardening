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

[sshd_config](configs/sshd/sshd_config) file with restrictions in place [R1]

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
chown -R <username>:<username> /home/<username>/.ssh
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

# Setting up a firewall [R6]

Verifying is ufw is active [R5]:

```sh
sudo ufw status
```

If UFW is not installed, run:

```sh
sudo apt-get install ufw
```

By default, it is a good idea to DROP all connections coming from the external world except for the few connections you want to explicitly allow.

```sh
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Now we allow for SSH, HTTP and HTTPS:

```sh
ufw allow ssh
ufw allow http
ufw allow https
```

When everything looks good and we are confidend about our rules:

```sh
sudo ufw enable
```
Now we have a firewall :)

# Sending email

I want to send emails from my server. In this particular case I want to send fail2ban notifications.

If you are using UFW as a firewall you probably want to allow emails to be sent from the machine:

```sh
ufw allow out 587
```

Now we install sendmail:

```sh
apt-get install sendmail
```

# Installing fail2ban

Fail2ban keeps track of attemps to log into SSH on your server. Once it passes a treshold it blocks that IP. It can probably do more, but that's enough for me now. It seems like the default configurations are pretty good as they are so it's just a matter of installing it.

```sh
sudo apt-get install fail2ban
```

# References

- [R1] - https://linux-audit.com/audit-and-harden-your-ssh-configuration/
- [R2] - https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
- [R3] - https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
- [R4] - https://linux-audit.com/using-ssh-keys-instead-of-passwords/
- [R5] - https://askubuntu.com/questions/533269/how-to-check-if-ufw-is-running-programmatically
- [R6] - https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04
- [R7] - https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
- [R8] - https://www.linode.com/docs/security/using-fail2ban-for-security/
