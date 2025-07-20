Hey Guys...

Welcome back again, in this post I will be guiding you on how to disable IPv6 on your Ubuntu box.

**Step 1:** login to the Ubuntu terminal.

**Step 2:** use command _ifconfig _to check the status of IPv6 on the machine.

If the IPv6 address is assigned then the output is as shown below.

[![u2]({{%20site.baseurl%20}}/assets/2014/07/u2.png)](https://shivamshukla.wordpress.com/wp-content/uploads/2014/07/u2.png)

**Step 3:** Now to disable the IPv6 use the below command.

Edit the _/etc/sysctl.conf _file by the command

_sudo vi /etc/sysctl.conf or sudo gedit _ /etc/sysctl.conf__

Add the below lines to the end of the file & save it.

_net.ipv6.conf.all.disable_ipv6 = 1_

_net.ipv6.conf.default.disable_ipv6 = 1_

_net.ipv6.conf.lo.disable_ipv6 = 1_

After saving the file use the below command to make the new settings effective.

_sudo sysctl –p_

**Step 4:** After disabling the IPv6 cross-check the status by the command _ifconfig._

The sample output is shown below.

[![u5]({{%20site.baseurl%20}}/assets/2014/07/u5.png)](https://shivamshukla.wordpress.com/wp-content/uploads/2014/07/u5.png)

hope you will find this post useful..

Thanks...