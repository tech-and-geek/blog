Hi Guys.

This post is about MySQL error "Cannot proceed because system tables used by Event Scheduler were found damaged at server start". 

This error was encountered by me while trying to upgrade the MySQL version  in CentOS

The reason for the upgrade being the old MySQL version was not having support for event scheduling.

After the upgrade was completed when I tried to create the events table the error popped up.

Here is work around which worked out for me.

**Step 1:** open the terminal

**Step 2:** execute the command 

```bash
_mysql_upgrade -p --force_

**Step 3:** after the second step execute this command 

```bash
_service mysql restart_
```
or 
```bash
_service mysqld restart_ 
```
now try to recreate the events table that should work out now.

Please do let me know if you found this post useful.

Thanks...