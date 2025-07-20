Hey guys here's a script to help you install MRTG+RRD on a CentOS box

The script has been tested on  CentOS 6.4 (Final) Desktop, you can use this script to install on a Minimal Desktop or Minimal system as well.

If you choose to install MRTG+RRD on a Minimal Desktop or Minimal version you may face issue with fonts during Graph creation. The fonts issue may get resolved by itself or if not then you might have to install font packages manually that is not included into the script.

**Note:**

**1) SELniux will be disabled if you run this script.**

**2) Port 80 (HTTP) will be allowed or IPTables would be disabled as per your choice. If you are running HTTP service on some custom port make sure to allow that manually if you choose not to disable IPTables.**

```bash
#!/bin/bash

#########################################

##Author: Shivam Shukla ##

##Build Date: 26-10-2014 ##

##Modification Date: 03-11-2014 ##

##Version: 1.0.0 ##

##Tool: MRTG+RRD Installer for CentOS ##

#########################################

##### User Variable #####

ipadd=`ifconfig |grep inet | cut -b 21-32 |head -n1`

#### User Functions####

function fn_SELinux()

{

getenforce |grep Enforcing >/dev/null 2>&1 && se_l=1 || se_l=0

if [ $se_l -eq 1 ]

then

echo "SELinux is Enforcing..."

echo "Disabling SELinux...."

setenforce 0

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

echo "SELinux is Disabled...:)"

else

echo "SELinux is Disabled :)"

fi

}

function fn_IPTable()

{

/etc/init.d/iptables status |grep not && ipt=1 || ipt=0

if [ $ipt -eq 1 ]

then

echo "IPtables is already disabled...."

else

echo ""

echo "IPtables service is running on the system..."

echo ""

echo "If you want I can stop the IPtable service for you or if you don't want to stop the IPtable service i will allow the httpd service in IPtables."

echo""

echo "Do you want me to stop the IPtable service [y/n]..."

read response

fi

echo "$response" > respo; head -1 respo > res

cat res | grep y >/dev/null 2>&1 && sure=1 || sure=0

if [ $sure -eq 1 ]

then

echo "Disabling IPTables..... "

/etc/init.d/iptables stop

chkconfig iptables off

echo "IPTables service is disabled now...:)"

rm -f res respo

else

echo "Checking if http service is allowed in IPtable..."

iptables -L |grep http >/dev/null 2>&1 && ena=1 || ena=0

if [ $ena -eq 1 ]

then

echo "http service is already allowed in IPtable :)"

else

echo "Adding httpd service to iptables..."

iptables -I INPUT 5 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

/etc/init.d/iptables save

echo "httpd service has been allowed in IPtables... :)"

rm -f res respo

fi

fi

}

####### Information Collection #######

echo "Please enter the network IP you want to add example 192.168.1.0/24"

read netwo

echo "please enter the state/province name where system is located"

read state

echo "Please enter the country name where system is located"

read country

echo "Please enter the system administrators name"

read contact

echo "Please enter the system administrators email"

read contactmail

####### Preparing the server for MRTG installation ######

echo "Checking prerequisites...."

fn_SELinux;

fn_IPTable;

echo "Checking for wget package..."

wget |grep Usage >/dev/null 2>&1 && wgt=1 || wgt=0

if [ $wgt -eq 1 ]

then

echo "wget is already installed"

else

echo "Installing wget"

yum install -y wget

fi

echo "Installing other required packages..."

###### Necessary Package Installation ######

yum install -y httpd gcc gd gd-devel perl libpng libxml2-devel cairo-devel glib2-devel pango-devel perl-devel perl-CGI net-snmp net-snmp-utils

###### SNMP Configuration #######

mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig

echo "com2sec local localhost public

com2sec mynetwork $netwo public

group MyRWGroup v1 local

group MyRWGroup v2c local

group MyRWGroup usm local

group MyROGroup v1 mynetwork

group MyROGroup v2c mynetwork

group MyROGroup usm mynetwork

view all included .1 80

access MyROGroup \"\" any noauth exact all none none

access MyRWGroup \"\" any noauth exact all all none

syslocation $state, $country

syscontact $contact <$contactmail>

" > /etc/snmp/snmpd.conf

/etc/init.d/snmpd restart

chkconfig snmpd on

##### MRTG Installation #######

cd /tmp

wget http://oss.oetiker.ch/mrtg/pub/mrtg.tar.gz

tar zfx mrtg.tar.gz >/dev/null 2>&1

tar -ztf mrtg.tar.gz |head -n1 |awk -F "/THANKS" '{print $1}' > mrtgvers

mrtver="$(cat /tmp/mrtgvers)"

cd $mrtver

./configure --prefix=/usr/local/mrtg2

make

make install

cd /tmp

wget http://oss.oetiker.ch/rrdtool/pub/rrdtool.tar.gz

tar zfx rrdtool.tar.gz >/dev/null 2>&1

tar -ztf rrdtool.tar.gz |head -n1 |awk -F "/THANKS" '{print $1}' > rrdvers

rrdver="$(cat /tmp/rrdvers)"

cd $rrdver

./configure --prefix=/usr/local/rrdtool

make

make install

mkdir /var/www/mrtg

mkdir /home/mrtg

/usr/local/mrtg2/bin/cfgmaker --global "WorkDir: /var/www/mrtg" --global "Options[_]: growright, bits" --global "RunAsDaemon: Yes" --global "LogFormat: rrdtool" --global "PathAdd: /usr/local/rrdtool/bin" --global "LibAdd: /usr/local/rrdtool/lib/perl/5.10.1" -ifref=ip --output /home/mrtg/mrtg.cfg public@localhost

env LANG=C /usr/local/mrtg2/bin/mrtg /home/mrtg/mrtg.cfg

echo "*/5 * * * * root env LANG=C /usr/local/mrtg2/bin/mrtg /home/mrtg/mrtg.cfg --logging /var/log/mrtg.log " > /etc/cron.d/mrtg

chkconfig crond on

/usr/local/mrtg2/bin/indexmaker --output=/var/www/mrtg/index.html /home/mrtg/mrtg.cfg

echo "

# This configuration file maps the mrtg output (generated daily)

# into the URL space. By default these results are only accessible

# from the local host.

#

Alias /mrtg /var/www/mrtg

<Location /mrtg>

Order deny,allow

#Deny from all

Allow from 127.0.0.1

Allow from ::1

# Allow from .example.com

Allow from all

</Location>

" > /etc/httpd/conf.d/mrtg.conf

chkconfig httpd on

service httpd restart

cd /tmp

wget https://shivamshukla.wordpress.com/wp-content/uploads/2014/10/14all-cgi1.key >/dev/null 2>&1

mv /tmp/14all-cgi1.key /var/www/cgi-bin/14all.cgi

chmod 777 /var/www/cgi-bin/14all.cgi

echo "MRTG Installation Completed.... :)"

echo "Click the link to open the MRTG page http://$ipadd/mrtg"


```

[**Click here to download the script**](https://shivamshukla.wordpress.com/wp-content/uploads/2014/11/mrtg.key)



