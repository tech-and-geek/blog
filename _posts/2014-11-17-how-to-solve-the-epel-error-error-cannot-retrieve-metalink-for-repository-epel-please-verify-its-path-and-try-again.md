Hi Guys,

I have been working on the EPEL repository these day's & at times I got around with the following error.

**Error: Cannot retrieve metalink for repository: epel. Please verify its path and try again**

```bash
[root@localhost]# yum list

Loaded plugins: fastestmirror

Determining fastest mirrors

Error: Cannot retrieve metalink for repository: epel. Please verify its path and try again

[root@localhost]


```

Here's how you can solve this error if you come across this error.

edit the file epel.repo which is located in the /etc/yum.repos.d/ using your favorite editor.

& change the https link to http as show below.

```bash
[root@localhost]# vi /ete/yum.repos.d/epel.repo


```

now change the https to http

```bash
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch


```

to

```bash
mirrorlist=http://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch


```

do this for all the links present in  the file.

then check the yum again

```bash
[root@localhost]# yum list

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

* base: mirror.nbrc.ac.in

* epel: mirrors.123host.vn

* extras: mirror.nbrc.ac.in

* rpmforge: archive.cs.uu.nl

.

.


```

That's it the issue is solved

hope you find this post helpful please don't forget to share you feedback or comments about my post.