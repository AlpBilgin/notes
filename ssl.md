# ssl - tls

## make a self signed cert with open ssl

run this in ~
 
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./.ssh/pxy-selfsigned.key -out ./.ssh/pxy-selfsigned.crt
```