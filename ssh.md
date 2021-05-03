# Server Side



## Terminal generic
ssh-copy-id -i ~/.ssh/id_rsa.pub YOUR_REMOTE_USER_NAME@IP_ADDRESS_OF_THE_SERVER to send sh key

## ubuntu

install openssh in linux, sudo apt install openssh-server
check if it runs sudo systemctl status ssh
systemctl start ssh to start
 ubuntu:: sudo ufw allow ssh to puch hole in firewall

## MacOS

- /private/etc/ssh/ is where you find the important OS wide settings
- /private/etc/ssh/sshd_cinfig is where host settings are kept
- /var/empty should be empty
- assuming environment is broken enough that sshd is not running: $(which sshd) -d will try to start sshd and give you a debug trace.

# Client Side

## Terminal generic

### generate new key, github default

`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
id_rsa.pub is the password to register with servers

### when working with bare metal servers, programmatically send add your ssh key to server that you can access

`cat ~/.ssh/id_rsa.pub | ssh username@server.dreamhost.com "cat >> ~/.ssh/authorized_keys"`

you may have to add a mkdir in the quotes to create the .ssh dir.

### remove outdated keys MAKE SURE TARGET SERVER IS NOT COMPROMISED

ssh-keygen -f "/home/<username>/.ssh/known_hosts" -R "hostname|ip"

## Linux

###when working with bare metal servers, programmatically send add your ssh key to server that you can access. Less error prone than generic solution.

`ssh-copy-id -i ~/.ssh/id_rsa.pub username@serverurl`

