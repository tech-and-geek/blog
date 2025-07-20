System details:-


OpenVPN Server IP: 192.168.1.2 (Public IP, the written IP is private used for example purpose)


MySQL Server IP: 192.168.1.2 (Public IP, the written IP is private used for example purpose)


Shell script (Customize)- 1 user - many connections


Install MySQL Server for User/Pass Authentication, IP = 192.168.1.2


Install MySQL Server

```bash
[root@localhost]# yum install mysql-server
```

Log in MySQL as root
```bash
[root@localhost]#mysql -uroot -p
```

Create the database 'openvpn'

```bash
mysql> CREATE DATABASE openvpn;
```

Create a MySQL user with username 'USERNAME' and password 'PASSWORD'
```bash
mysql>GRANT ALL ON openvpn.* TO 'USERNAME'@"%" IDENTIFIED BY 'PASSWORD';
```

Log out root user
```bash
mysql>exit
```

Log in MySQL as new user 'USERNAME'
```bash
[root@localhost]# mysql -uUSERNAME -pPASSWORD
```

Switch database
```bash
mysql>USE openvpn;
```

Create the user, log table and insert user data into it.
User table creation
```bash

CREATE TABLE IF NOT EXISTS `user` (
    `user_id` varchar(32) COLLATE utf8_unicode_ci NOT NULL,
    `user_pass` varchar(32) COLLATE utf8_unicode_ci NOT NULL DEFAULT '1234',
    `user_mail` varchar(64) COLLATE utf8_unicode_ci DEFAULT NULL,
    `user_phone` varchar(16) COLLATE utf8_unicode_ci DEFAULT NULL,
    `user_online` tinyint(1) NOT NULL DEFAULT '0',
    `user_enable` tinyint(1) NOT NULL DEFAULT '1',
    `user_start_date` date NOT NULL,
    `user_end_date` date NOT NULL,
PRIMARY KEY (`user_id`),
KEY `user_pass` (`user_pass`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

Log table creation
```bash

CREATE TABLE IF NOT EXISTS `log` (
    `log_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    `user_id` varchar(32) COLLATE utf8_unicode_ci NOT NULL,
    `log_trusted_ip` varchar(32) COLLATE utf8_unicode_ci DEFAULT NULL,
    `log_trusted_port` varchar(16) COLLATE utf8_unicode_ci DEFAULT NULL,
    `log_remote_ip` varchar(32) COLLATE utf8_unicode_ci DEFAULT NULL,
    `log_remote_port` varchar(16) COLLATE utf8_unicode_ci DEFAULT NULL,
    `log_start_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `log_end_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
    `log_received` float NOT NULL DEFAULT '0',
    `log_send` float NOT NULL DEFAULT '0',
PRIMARY KEY (`log_id`),
KEY `user_id` (`user_id`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

User data insertion
```bash

INSERT INTO `user` (
    `user_id`, `user_pass`, `user_mail`, `user_phone`,
    `user_online`, `user_enable`, `user_start_date`, `user_end_date`
)
VALUES (
    'foobar', 'foo@123', 'foo.bar@foobar.com',
    '+1234567890', 0, 1, '2014-01-01', '0000-00-00'
);```

Now let's have a look at the tables &; user data we just created.
Tabel view
```bash
mysql>show tables;
+-------------------+
| Tables_in_openvpn |
+-------------------+
| log               |
| user              |
+-------------------+
```

User data view
```bash
mysql>select * from user;
+---------+-----------+---------------------+--------------+-------------+-------------+-----------------+---------------+
| user_id | user_pass | user_mail           | user_phone   | user_online | user_enable | user_start_date | user_end_date |
+---------+-----------+---------------------+--------------+-------------+-------------+-----------------+---------------+
| foobar    | foo@123      | foo.bar@foobar.com | +1234567890 |           0 |           1 | 2014-01-01      | 0000-00-00    |
+---------+-----------+---------------------+--------------+-------------+-------------+-----------------+---------------+
```

Everything seems to be OK now let's logout of mysql
```bash
mysql>exit
```

Edit file /etc/mysql/my.cnf using your favourite editor &; insert # to line
```bash
[root@localhost]# vi /etc/my.cnf
bind-address  = 127.0.0.1
```

Sample
```bash
#bind-address  = 127.0.0.1
```

OpenVPN Server Side Configuration
Prerequisites
OpenVPN and it’s dependencies are not available in CentOS default repositories. So, it is required to install the“EPEL” repo. in order to install OpenVPN and its dependencies.
Lets start with the installation of EPEL repo first. Downloading &; installing the rpm from the below link will enable the EPEL repo on your CentOS box.
## RHEL/CentOS 6 32-Bit ##
```bash

[root@localhost]# wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
[root@localhost]# rpm -ivh epel-release-6-8.noarch.rpm
```

## RHEL/CentOS 6 64-Bit ##
```bash

[root@localhost]# wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
[root@localhost]# rpm -ivh epel-release-6-8.noarch.rpm
```

Its better to update the system using the following command:
```bash

yum update
```

Install OpenVPN Software
Install the OpenVPN software using the following command:
```bash

yum install openvpn easy-rsa
```

The easy-rsa scripts by default are in the /usr/share/easy-rsa/ directory. Make a directory /easy-rsa/keys inside the /etc/openvpn directory and copy those scripts to that directory as shown below:
```bash

mkdir -p /etc/openvpn/easy-rsa/keys
cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa/
```

Generate CA Certificate and CA key
Edit file /etc/openvpn/easy-rsa/2.0/vars
```bash

[root@localhost]vi /etc/openvpn/easy-rsa/vars
```

And, change the values that matches with your country, state, city, mail id etc.

```bash

# Don't leave any of these fields blank.
export KEY_COUNTRY="IN"
export KEY_PROVINCE="Delhi"
export KEY_CITY="New Delhi"
export KEY_ORG="Test_Lab"
export KEY_EMAIL="foo@foobar.com"
export KEY_OU="server"
```

Go to the openvpn/easy-rsa directory:
```bash

cd /etc/openvpn/easy-rsa/
```

Enter the following commands one by one to initialize the certificate authority:
```bash

cp openssl-1.0.0.cnf openssl.cnfsource
source ./vars
./clean-all
```

Then, run the following command to generate CA certificate and CA key:
```bash

./build-ca
```

Sample output:
```bash

Generating a 2048 bit RSA private key
......................................................+++
............................................................+++
writing new private key to 'ca.key'
----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [IN]: ----> Press Enter
State or Province Name (full name) [DEL]:----> Press Enter
Locality Name (eg, city) [New Delhi]: ----> Press Enter
Organization Name (eg, company) [Test_Lab]: ----> Press Enter
Organizational Unit Name (eg, section) [server]: ----> Press Enter
Common Name (eg, your name or your server's hostname) [server CA]: ----> Press Enter
Name [EasyRSA]: ----> Press Enter
Email Address [foo@foobar.com]: ----> Press Enter


We have now generated the CA certificate and CA key. Then create certificate and key for server using the following command:


./build-key-server server
```

Sample output:
```bash
Generating a 2048 bit RSA private key
....................+++
.............+++
writing new private key to 'server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [IN]: ----> Press Enter
State or Province Name (full name) [DEL]: ----> Press Enter
Locality Name (eg, city) [New Delhi]: ----> Press Enter
Organization Name (eg, company) [Test_Lab]: ----> Press Enter
Organizational Unit Name (eg, section) [server]: ----> Press Enter
Common Name (eg, your name or your server's hostname) [server]: ----> Press Enter
Name [EasyRSA]: ----> Press Enter
Email Address [foo@foobar.com]: ----> Press Enter
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: ----> Press Enter
An optional company name []: ----> Press Enter
Using configuration from /etc/openvpn/easy-rsa/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'IN'
stateOrProvinceName   :PRINTABLE:'DEL'
localityName          :PRINTABLE:'New Delhi'
organizationName      :PRINTABLE:'Test_Lab'
organizationalUnitName:PRINTABLE:'server'
commonName            :PRINTABLE:'server'
name                  :PRINTABLE:'EasyRSA'
emailAddress          :IA5STRING:'foo@foobar.com'
Certificate is to be certified until Mar 23 12:21:34 2024 GMT (3650 days)
Sign the certificate? [y/n]:y ----> Type Y and Press Enter
1 out of 1 certificate requests certified, commit? [y/n]y ----> Type Y and Press Enter
Write out database with 1 new entries
Data Base Updated
```

Create certificate and key for VPN clients using the following command:
```bash
./build-key client
```

If you want to create certificate and key files for each client, you should replace the client parameter with an unique identifier.
Sample output:
```bash
Generating a 2048 bit RSA private key
.......+++
..................................................................................................+++
writing new private key to 'client.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [IN]: ----> Press Enter
State or Province Name (full name) [DEL]: ----> Press Enter
Locality Name (eg, city) [New Delhi]: ----> Press Enter
Organization Name (eg, company) [Test_Lab]: ----> Press Enter
Organizational Unit Name (eg, section) [server]:----> Press Enter
Common Name (eg, your name or your server's hostname) [client]: ----> Press Enter
Name [EasyRSA]: ----> Press Enter
Email Address [foo@foobar.com]: ----> Press Enter
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: ----> Press Enter
An optional company name []: ----> Press Enter
Using configuration from /etc/openvpn/easy-rsa/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'IN'
stateOrProvinceName   :PRINTABLE:'DEL'
localityName          :PRINTABLE:'New Delhi'
organizationName      :PRINTABLE:'Test_Lab'
organizationalUnitName:PRINTABLE:'server'
commonName            :PRINTABLE:'client'
name                  :PRINTABLE:'EasyRSA'
emailAddress          :IA5STRING:'foo@foobar.com'
Certificate is to be certified until Mar 23 12:23:44 2024 GMT (3650 days)
Sign the certificate? [y/n]:y ----> Type Y and Press Enter
1 out of 1 certificate requests certified, commit? ----> Type Y and Press Enter
Write out database with 1 new entries
Data Base Updated
```

Generate Diffie Hellman (dh) Parameter
Enter the following command to generate DH parameter.
```bash
./build-dh
```

Sample output:

```bash
Generating DH parameters, 2048 bit long safe prime, generator 2This is going to take a long time

The necessary keys and certificates will be generated in the /etc/openvpn/easy-rsa/keys/ directory. Copy the following certificate and key files to the /etc/openvpn/keys directory.

ca.crt
dh2048.pem
server.crt
server.key
```

Go to the directory /etc/openvpn/easy-rsa/keys/ and enter the following command to transfer the above files to/etc/openvpn/keys directory.
```bash
[root@localhost]# mkdir /etc/openvpn/keys
[root@localhost]#cd /etc/openvpn/easy-rsa/keys/
[root@localhost]#cp dh2048.pem ca.crt server.crt server.key /etc/openvpn/keys
```

And then, you must copy all client certificates and keys to the remote VPN clients in order to authenticate to the VPN server. In our case, we have generated certificates and keys to only one client, so we have to copy the following files to the VPN client.
```bash

ca.crt
client.crt
client.key
```

Now let's tuneup the setup to use user authorization parameters.
Customize shell script, IP = 192.168.1.2
Create directory for script '/etc/openvpn/script'
```bash
[root@localhost]#mkdir /etc/openvpn/script
[root@localhost]#cd /etc/openvpn/script
```

Create file config.sh '/etc/openvpn/script/config.sh' 
```bash
[root@localhost]vi /etc/openvpn/script/config.sh
```

Insert the below code into the config.sh file
```bash
#!/bin/bash
##Dababase Server
HOST='192.168.1.2'
#Default port = 3306
PORT='3306'
#Username
USER='USERNAME'
#Password
PASS='PASSWORD'
#database name
DB='openvpn'
```

Create file test_connect_db.sh'/etc/openvpn/script/test_connect_db.sh'
```bash
[root@localhost]# vi test_connect_db.sh
```

Insert the below code into the test_connect_db.sh file
```bash
#!/bin/bash
. /etc/openvpn/script/config.sh
##Test Authentication
username=$1
password=$2
user_id=$(mysql -h$HOST -P$PORT -u$USER -p$PASS $DB -sN -e "select user_id from user where user_id = '$username' AND user_pass = '$password' AND user_enable=1 AND user_start_date != user_end_date AND TO_DAYS(now()) >= TO_DAYS(user_start_date) AND (TO_DAYS(now()) &lt;= TO_DAYS(user_end_date) OR user_end_date='0000-00-00')")
##Check user
[ "$user_id" != '' ] &;&; [ "$user_id" = "$username" ] &;&; echo "user : $username" &;&; echo 'authentication ok.' &;&; exit 0 || echo 'authentication failed.'; exit 1
```
Create file login.sh '/etc/openvpn/script/login.sh'

```bash
[root@localhost]# vi /etc/openvpn/script/login.sh
```

Insert the below code into the login.sh file
```bash
#!/bin/bash
. /etc/openvpn/script/config.sh
##Authentication
user_id=$(mysql -h$HOST -P$PORT -u$USER -p$PASS $DB -sN -e "select user_id from user where user_id = '$username' AND user_pass = '$password' AND user_enable=1 AND user_start_date != user_end_date AND TO_DAYS(now()) >= TO_DAYS(user_start_date) AND (TO_DAYS(now()) &lt;= TO_DAYS(user_end_date) OR user_end_date='0000-00-00')")
##Check user
[ "$user_id" != '' ] &;&; [ "$user_id" = "$username" ] &;&; echo "user : $username" &;&; echo 'authentication ok.' &;&; exit 0 || echo 'authentication failed.'; exit 1
```

Create file connect.sh '/etc/openvpn/script/connect.sh'
```bash
[root@localhost]# vi /etc/openvpn/script/connect.sh```

Insert the below code into the file connect.sh
```bash
#!/bin/bash
. /etc/openvpn/script/config.sh
##insert data connection to table log
mysql -h$HOST -P$PORT -u$USER -p$PASS $DB -e "INSERT INTO log (log_id,user_id,log_trusted_ip,log_trusted_port,log_remote_ip,log_remote_port,log_start_time,log_end_time,log_received,log_send) VALUES(NULL,'$common_name','$trusted_ip','$trusted_port','$ifconfig_pool_remote_ip','$remote_port_1',now(),'0000-00-00 00:00:00','$bytes_received','$bytes_sent')"
##set status online to user connected
mysql -h$HOST -P$PORT -u$USER -p$PASS $DB -e "UPDATE user SET user_online=1 WHERE ser_id='$common_name'"
```

Create file disconnect.sh '/etc/openvpn/script/disconnect.sh'
```bash
[root@localhost]# vi /etc/openvpn/script/disconnect.sh
```

Insert the below code into the disconnect.sh file
```bash
#!/bin/bash
. /etc/openvpn/script/config.sh
##set status offline to user disconnected
mysql -h$HOST -P$PORT -u$USER -p$PASS $DB -e "UPDATE user SET user_online=0 WHERE user_id='$common_name'"
##insert data disconnected to table log
mysql -h$HOST -P$PORT -u$USER -p$PASS $DB -e "UPDATE log SET log_end_time=now(),log_received='$bytes_received',log_send='$bytes_sent' WHERE log_trusted_ip='$trusted_ip' AND log_trusted_port='$trusted_port' AND user_id='$common_name' AND log_end_time='0000-00-00 00:00:00'"
```

Compose OpenVPN configuration files, OpenVPN server will scan the .conf files in /etc/openvpn when it starts. For each file, it forks a daemon. In this system, we need both UDP and TCP support. I created two configuration files for two daemons in charge of UDP and TCP respectively.
Create file server-tcp-443.conf '/etc/openvpn/server-tcp-443.conf' for Server Port:443
```bash

##protocol port
port 443
proto tcp
dev tun
##ip server client
server 10.4.0.0 255.255.255.0
##key
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/server.crt
key /etc/openvpn/keys/server.key
dh /etc/openvpn/keys/dh1024.pem
##option
persist-key
persist-tun
keepalive 5 60
reneg-sec 432000
##option authen.
comp-lzo
user nobody
#group nogroup
client-to-client
username-as-common-name
client-cert-not-required
auth-user-pass-verify /etc/openvpn/script/login.sh via-env
##push to client
max-clients 50
push "persist-key"
push "persist-tun"
push "redirect-gateway def1"
#push "explicit-exit-notify 1"
##DNS-Server
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
##script connect-disconnect
script-security 3 system
client-connect /etc/openvpn/script/connect.sh
client-disconnect /etc/openvpn/script/disconnect.sh
##log-status
status /etc/openvpn/log/tcp_443.log
log-append /etc/openvpn/log/openvpn.log
verb 3
```

Create file server-udp-53.conf '/etc/openvpn/server-udp-53.conf' for Server Port:53
```bash
[root@localhost]# vi /etc/openvpnserver-udp-53.conf
```

Insert the below lines to the server-udp-53.conf
```bash
##protocol port
port 53
proto udp
dev tun
##ip server client
server 10.5.0.0 255.255.255.0
##key
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/server.crt
key /etc/openvpn/keys/server.key
dh /etc/openvpn/keys/dh1024.pem
##option
persist-key
persist-tun
keepalive 5 60
reneg-sec 432000
##option authen.
comp-lzo
user nobody
#group nogroup
client-to-client
username-as-common-name
client-cert-not-required
auth-user-pass-verify /etc/openvpn/script/login.sh via-env
##push to client
max-clients 50
push "persist-key"
push "persist-tun"
push "redirect-gateway def1"
push "explicit-exit-notify 1"
##DNS-Server
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
##script connect-disconnect
script-security 3 system
client-connect /etc/openvpn/script/connect.sh
client-disconnect /etc/openvpn/script/disconnect.sh
##log-status
status /etc/openvpn/log/udp_53.log
log-append /etc/openvpn/log/openvpn.log
verb 3
```

Create directory for log '/etc/openvpn/log'
```bash
[root@localhost]# mkdir /etc/openvpn/log
[root@localhost]# touch /etc/openvpn/log/openvpn.log
[root@localhost]# touch /etc/openvpn/log/tcp_443.log
[root@localhost]# touch /etc/openvpn/log/udp_53.log
```

Changes the permission of files
```bash
chmod -R 755 /etc/openvpn
```

Test authentication username 'foobar' and password 'foo@123'
```bash
[root@localhost]# /etc/openvpn/script/test_connect_db.sh foobar foo@123
# user : test
# authentication ok.
# if authentication failed. check user and password in database
# or detail database server in /etc/openvpn/script/config.sh
```

Start serviece OpenVPN
```bash
[root@localhost]# /etc/init.d/openvpn start
```

Edit file /etc/sysctl.conf Remove # In line : #net.ipv4.ip_forward=1
```bash
[root@localhost]#vi /etc/sysctl.conf
net.ipv4.ip_forward=1
```

Execute the sysctl -p command for the changes to take effect
```bash
[root@localhost]# sysctl -p
```

Edit file /etc/rc.local Add before exit 0; the below code
```bash

echo "1" > /proc/sys/net/ipv4/ip_forward
echo "1" > /proc/sys/net/ipv4/ip_dynaddr
iptables -A INPUT -i tun0 -j ACCEPT
iptables -A FORWARD -i tun0 -j ACCEPT
iptables -A INPUT -i tun1 -j ACCEPT
iptables -A FORWARD -i tun1 -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp -m tcp --dport 3306 -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.4.0.0/24 -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.5.0.0/24 -o eth0 -j MASQUERADE
iptables -A INPUT -i eth0 -p tcp -m tcp --dport 3306 -j ACCEPT
```

Run Script Iptables
```bash
[root@localhost]# /etc/rc.local
[root@localhost]# iptables-save
```

Config for Client
Config for port TCP port 443
```bash

client
dev tun
proto tcp
remote 192.168.1.2 443
nobind
auth-user-pass
reneg-sec 432000
resolv-retry infinite
ca ca.crt
comp-lzo
verb 1
```

Config for port UDP port 53
```bash

client
dev tun
proto udp
remote 1.1.1.1 53
nobind
auth-user-pass
reneg-sec 432000
resolv-retry infinite
ca ca.crt
comp-lzo
verb 1
```

That's it, all done
Test it &; share your feedback.  :)