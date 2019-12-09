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

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
id_rsa.pub is the password to send to servers
