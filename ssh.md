# Server Side

## Terminal generic

## MacOS

- /private/etc/ssh/ is where you find the important OS wide settings
- /private/etc/ssh/sshd_cinfig is where host settings are kept
- /var/empty should be empty
- assuming environment is broken enough that sshd is not running: $(which sshd) -d will try to start sshd and give you a debug trace.

# Client Side

## Terminal generic

###generate new key, github default

`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
id_rsa.pub is the password to register with servers

###when working with bare metal servers, programmatically send add your ssh key to server that you can access

`cat ~/.ssh/id_rsa.pub | ssh username@server.dreamhost.com "cat >> ~/.ssh/authorized_keys"`

you may have to add a mkdir in the quotes to create the .ssh dir.

## Linux

###when working with bare metal servers, programmatically send add your ssh key to server that you can access. Less error prone than generic solution.

`ssh-copy-id -i ~/.ssh/id_rsa.pub username@serverurl`

