Hey guys,

Hope you are having fun these day's  today we are going to try on the EPEL repository installation on CentOS 6.

For 32/64 Bit Operating system.

use the below link & install it as show below.

http://dl.fedoraproject.org/pub/epel/6Server/i386/epel-release-6-8.noarch.rpm

```bash

[root@localhost]# rpm -ivh http://dl.fedoraproject.org/pub/epel/6Server/i386/epel-release-6-8.noarch.rpm

Retrieving http://dl.fedoraproject.org/pub/epel/6Server/i386/epel-release-6-8.noarch.rpm

Preparing...                ########################################### [100%]

1:epel-release           ########################################### [100%]

[root@localhost]#
```

Now lets verify weather our repository has been installed or not

```bash


[root@localhost]# rpm –qa |grep epel

epel-release-6-8.noarch

[root@localhost]


```

yeah that's all the installation is complete.

Enjoy :)