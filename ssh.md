# Server Side

## Terminal generic

## MacOS

/private/etc/ssh/ is where you find the important OS wide settings
/private/etc/ssh/sshd_cinfig is where host settings are kept

# Client Side

## Terminal generic

###generate new key, github default

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
id_rsa.pub is the password to send to servers
