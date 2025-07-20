Hey Guy's

Here my new post about the Google Authenticator, its really a very good & effective tool.

Hope you will find this post useful.

Google Two-Factor Authentication provides next level of security from intruders to SSH server. This article will help you understand the process & how to protect your SSH server with a two-factor authentication using Google Authenticator & PAM module. Now Every time when you try to SSH to your server, you have to generate code using your phone or other devices to login the server.



**Step 1:** EPEL Repository Installation

First we need to add EPEL yum repository on the system. Follow this link on how to install EPEL repository.

[**Click here to view the setup guide**](https://shivamshukla.wordpress.com/2014/11/17/how-to-install-epel-repository-on-centos-6/)


then proceed ahead with the step 2



**Step 2:** Google Authenticator Installation

Install Google Authenticator using yum command line tool.

```bash
[root@localhost]# yum install google-authenticator

```


**Step 3:** Configure Google authenticator

For this tutorial, we will use demo account for testing.

Use below steps to configure google-authenticator for user demouser.

```bash


[root@localhost]# su - demouser

[demouser@localhost]$ google-authenticator


 ```

now you will get the secret key & emergency codes please write them down & keep them safe. Emergency codes can only be used one time in case if your secret key is lost.

```bash

https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/demouser@localhost%3Fsecret%3DW2CUM37JJKN2KF2S

Your new secret key is: W2CUM37JJKN2KF2S

Your verification code is 357458

Your emergency scratch codes are:

56071230

90988902

61142941

30330862

64907016


 ```

After the secret codes the authenticator will ask few questions respond to them as yes if you don't want any specific changes.

```bash


Do you want me to update your "~/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication

token? This restricts you to one login about every 30s, but it increases

your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, tokens are good for 30 seconds and in order to compensate for

possible time-skew between the client and the server, we allow an extra

token before and after the current time. If you experience problems with poor

time synchronization, you can increase the window from its default

size of 1:30min to about 4min. Do you want to do so (y/n) y

If the computer that you are logging into isn't hardened against brute-force

login attempts, you can enable rate-limiting for the authentication module.

By default, this limits attackers to no more than 3 login attempts every 30s.

Do you want to enable rate-limiting (y/n) y

[demouser@localhost]$


 ```

Use Google Authenticator Application in your Android, iPhone or Blackberry phones to generate verification code by entering secret key. You can also use the add on available for firefox browser.

After installing the application on your phone here's how you can add the details on it.

Select "Enter provided key" option

[![gsnap1]({{%20site.baseurl%20}}/assets/2014/11/gsnap1.png)](https://shivamshukla.wordpress.com/wp-content/uploads/2014/11/gsnap1.png)

 

Enter the details required here like demouser & the secret key

[![gsnap2]({{%20site.baseurl%20}}/assets/2014/11/gsnap2.png)](https://shivamshukla.wordpress.com/wp-content/uploads/2014/11/gsnap1.png)

 

After adding the details you will get the vitrification codes here.

[![gsnap3]({{%20site.baseurl%20}}/assets/2014/11/gsnap3.png)](https://shivamshukla.wordpress.com/wp-content/uploads/2014/11/gsnap1.png)

**Step 4:** Activate Google authenticator

To enable google authenticator edit **/etc/pam.d/sshd** using your favorite editor.

```bash
[root@localhost]# vi /etc/pam.d/sshd
 ```

Add the below line to the start of the file just below the first line

```bash
auth required pam_google_authenticator.so
 ```

Now edit **/etc/ssh/sshd_config**  and Change **ChallengeResponseAuthentication** option value to **‘yes’**. On enabling this, openssh could ask a user any number of multi-facited (Like google authenticator) questions. Generally the system asks only for the user’s password.

```bash
[root@localhost]# vi /etc/ssh/sshd_config
 ```

Change the value from NO to YES

```bash
ChallengeResponseAuthentication yes
 ```

Restart the SSH service

```bash
[root@localhost]# service sshd restart
 ```

Now SSH your server & you will be asked for verification code first then the password.

Note: you will have to generate the secret code for each user separately & add them separately on application as well.

have fun let & me know your suggestions.

:)