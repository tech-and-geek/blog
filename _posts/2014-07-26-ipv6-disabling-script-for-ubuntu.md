Hey Guys...

Hope you are doing well here's a small script to help you to disable IPv6 on your Ubuntu Box.

hope you will find this helpful, let me know if you face any issue with this.

 

```bash
#########################################

##Author: Shivam Shukla ##

##Build Date: 22-07-2014 ##

##Modification Date: NA ##

##Version: 1.0.0 ##

##Tool: Ubuntu IPv6 Disabler ##

#########################################

#!/bin/bash

clear

int=$(netstat -i | cut -d" " -f1 | egrep -v "^Kernel|Iface|lo|:" |head -1)

ip -6 addr show dev $int |cut -b 5-9 >/tmp/ipp

grep "inet6" /tmp/ipp >/dev/null 2>&1 && ip_v=1 || ip_v=0

i6=$(ip addr show dev $int | sed -e's/^.*inet6 \([^ ]*\)\/.*$/\1/;t;d')

if [ $ip_v -eq 1 ]

then

echo "echo IPv6 found...";

echo "The IPv6 address is $i6";

echo "Disabling IPv6 address...";

echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf

echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf

echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf

sysctl -p

echo "IPv6 address disable success... :)"

else

echo "IPv6 is already disabled."

fi

rm -f /tmp/ipp
```