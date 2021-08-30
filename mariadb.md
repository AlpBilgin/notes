# mariadb

## install
### fetch bins
sudo apt-get install mariadb-server mariadb-client
### setup script
 mysql_secure_installation

### generate cert
 cd /etc/mysql
$ sudo mkdir ssl
$ cd ssl

Note: Common Name value used for the server and client certificates/keys must each differ from the Common Name value used for the CA certificate. To avoid any issues, I am setting them as follows. Otherwise, you will get certification verification failed error. Hence set it as follows:
CA common Name : MariaDB admin
Server common Name: MariaDB server
Client common Name: MariaDB client

sudo openssl genrsa 4096 > ca-key.pem

 sudo openssl req -new -x509 -nodes -days 365000 -key ca-key.pem -out ca-cert.pem

 creates following files

 /etc/mysql/ssl/ca-cert.pem – Certificate file for the Certificate Authority (CA).
/etc/mysql/ssl/ca-key.pem – Key file for the Certificate Authority (CA).

generae serv key

sudo openssl req -newkey rsa:2048 -days 365000 -nodes -keyout server-key.pem -out server-req.pem

generate rsa key

sudo openssl rsa -in server-key.pem -out server-key.pem

sign cert

sudo openssl x509 -req -in server-req.pem -days 365000 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem

Now you must have additional files:

/etc/mysql/ssl/server-cert.pem – MariaDB server certificate file.
/etc/mysql/ssl/server-key.pem – MariaDB server key file.
You must use above two files on MariaDB server itself and any other nodes that you are going to use for cluster/replication traffic. These two files will secure server side communication.

The mysql client, PHP/Python/Perl/Ruby app is going to use the client certificate to secure client-side connectivity. You must install the following files on all of your clients including the web server. To create the client key, run:
$ sudo openssl req -newkey rsa:2048 -days 365000 -nodes -keyout client-key.pem -out client-req.pem

Next, process client RSA key, enter:
$ sudo openssl rsa -in client-key.pem -out client-key.pem

Finally, sign the client certificate, run:
$ sudo openssl x509 -req -in client-req.pem -days 365000 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem

Type the following command to verify the certificates to make sure everything was created correctly:
$ openssl verify -CAfile ca-cert.pem server-cert.pem client-cert.pem


get the rest from  https://www.cyberciti.biz/faq/how-to-setup-mariadb-ssl-and-secure-connections-from-clients/
