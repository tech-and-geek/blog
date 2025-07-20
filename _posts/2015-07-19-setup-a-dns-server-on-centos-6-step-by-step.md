Hi Guys, here's my new post on how to set up a DNS server on a CentOS 6 box. Hope you will find this helpful.

### Scenario:

Here is my test setup scenario:

**DNS Server Details:**

| Setting         | Value                          |
|----------------|--------------------------------|
| OS             | CentOS release 6.4 (Final) 64-Bit |
| Hostname       | dnsserver                      |
| IP Address     | 192.168.1.26/24                |
| Domain Name    | localdnstest.local             |


> **Note**: The IP address used here is a private IP and the domain name is for testing purposes only.



Step 1: Install the packages required for DNS setup
```bash
[root@dnsserver ~ ]#yum -y install bind 
```
Step 2: Edit the "named.conf" file which is present in the "/etc/" directory the file looks somewhat like the one shown below, Edit/add the entries marked with comment to your file, make sure to use the details as per your requirement.
```bash
[root@dnsserver ~ ]#vi /etc/named.conf
```
```bash

// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
options {
 listen-on port 53 { 127.0.0.1; 192.168.1.26;}; #ADD/EDIT THE SERVER IP EXAMPLE 192.168.1.26
 listen-on-v6 port 53 { ::1; };
 directory "/var/named";
 dump-file "/var/named/data/cache_dump.db";
 statistics-file "/var/named/data/named_stats.txt";
 memstatistics-file "/var/named/data/named_mem_stats.txt";
 allow-query { localhost; 192.168.1.0/24;}; #ADD/EDIT THE IP RANGE HERE AS SHOWN
 recursion yes;
 dnssec-enable yes;
 dnssec-validation yes;
 dnssec-lookaside auto;
 /* Path to ISC DLV key */
 bindkeys-file "/etc/named.iscdlv.key";
 managed-keys-directory "/var/named/dynamic";
};
logging {
 channel default_debug {
 file "data/named.run";
 severity dynamic;
 };
};
zone "." IN {
 type hint;
 file "named.ca";
};
#ADD THE FORWARD &amp; REVERSE ZONE AS SHOWN BELOW
########Forward Zone#########################
zone "localdnstest.local" IN {
        type master;
        file "fwd.localdnstest.local";
        allow-update { none; };
};
#############################################
########Reverse Zone#######################
zone "1.168.192.in-addr.arpa" IN {
       type master;
       file "rev.localdnstest.local";
       allow-update { none; };
};
###########################################
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
Step 3: Creating zone files.
We need to create forward &amp; reverse zone files which we mentioned in the "/etc/named.conf" file.
Create the forward zone file "fwd.localdnstest.local" in the "/var/named" and add the entries for the forward zone as shown below.
```bash
[root@dnsserver ~ ]#vi /var/named/fwd.localdnstest.local
```
Add the below lines make sure to modify them as per your requirement.
```bash
$TTL 600
@ IN SOA  dnsserver.localdnstest.local. root.localdnstest.local. (
                                         2015022609  ; serial
                                         3600    ; refresh
                                         1800    ; retry
                                         604800  ; expire
                                         600 )   ; minimum
                IN NS   dnsserver.localdnstest.local.
                IN MX   10 mail.localdnstest.local.
dnsserver       IN A    192.168.1.26
mail            IN A    192.168.1.26
www             IN CNAME        dnsserver
ftp             IN CNAME        dnsserver
```
Create the reverse zone file "rev.localdnstest.local" in the "/var/named" and add the entries for the reverse zone as shown below.
```bash
[root@dnsserver ~ ]#vi /var/named/rev.localdnstest.local
```
Add the below lines make sure to modify them as per your requirement.
```bash
$TTL 600
@ IN SOA dnsserver.localdnstest.local. root.localdnstest.local. (
                               2015022609 ; serial
                               3600 ; refresh
                               1800 ; retry
                               604800 ; expire
                               600 ) ; minimum
            IN NS dnsserver.localdnstest.local.
            IN MX 10 mail.localdnstest.local.
dnsserver   IN A 192.168.1.26
mail        IN A 192.168.1.26
www         IN   CNAME    dnsserver
ftp         IN   CNAME    dnsserver
26          IN   PTR      dnsserver.localdnstest.local.
```
Step 4: After the creation of zone files, we need to start/restart the bind service. and allow it to start automatically.
```bash
[root@dnsserver ~ ]#service named start
[root@dnsserver ~ ]#chkconfig named on
```
Step 5: Allow the DNS sever through IP tables.
Edit the file "/etc/sysconfig/iptables" and add the below lines to the file
```bash
[root@dnsserver ~ ]#vi /etc/sysconfig/iptables
```
```bash
-A INPUT -p udp -m state --state NEW --dport 53 -j ACCEPT
-A INPUT -p tcp -m state --state NEW --dport 53 -j ACCEPT
```
Step 6: Restart the iptables service to save the changes.
```bash
[root@dnsserver ~ ]#service iptables restart
```
Step 7: Now let's test the DNS configuration &amp; zone files for syntax errors.
Check DNS configuration files.
```bash
[root@dnsserver ~ ]#named-checkconf /etc/named.conf
[root@dnsserver ~ ]#named-checkconf /etc/named.rfc1912.zones
```
Check zone files.
```bash
[root@dnsserver ~ ]#named-checkzone localdnstest.local /var/named/fwd.localdnstest.local
[root@dnsserver ~ ]#named-checkzone localdnstest.local /var/named/rev.localdnstest.local
```
NOTE: As this is a test domain so this test my not work until you modify the "/etc/resolv.conf" file and add the new nameserver parameter "nameserver 192.168.1.26"

```bash
[root@dnsserver ~ ]# vi /etc/resolve.conf
```

Add the below parameter.

```bash
nameserver 192.168.1.26
```

Step 8: Test the DNS server.

Method 1:
```bash
[root@dnsserver ~ ]#dig localdnstest.local
```

Method 2:
```bash
[root@dnsserver ~ ]#dig -x 192.168.1.26
```

Method 3:
```bash
[root@dnsserver ~ ]#nslookup www.localdnstest.local
```

Hope you find this interesting.
Thanks
