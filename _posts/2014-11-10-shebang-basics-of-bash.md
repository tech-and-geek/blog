Hi guys here's my new post about Shebang & some basics about BASH & creation of first bash script.


**What is Shebang?**
This symbol **#!** (Hash with Exclamation mark) is known as **shebang**,


**What Shebang does?**
It tells the shell what program to interpret the script with, when executed.


**Few** **examples**:


**#!/bin/bash**                   — Execute the file using bash, the Bourne again shell, or a compatible shell


**#!/bin/sh**                       — Execute the file using sh, the Bourne shell, or a compatible shell


**#!/bin/csh**                     — Execute the file using csh, the C shell, or a compatible shell


**#!/usr/bin/perl –T**         — Execute using Perl with the option for taint checks


**#!/usr/bin/php**              — Execute the file using the PHP command line interpreter


**#!/usr/bin/python -O**    — Execute using Python with optimizations to code


**#!/usr/bin/ruby**             — Execute using Ruby
 
Some commands to learn more about the shell you have on your system.
To check the shell type & path

```bash
#echo $SHELL
/bin/bash

```
OR
```bash
#which bash
/bin/bash

```
To check if the shell is interactive or non-interactive
```bash
#echo $-
himBH
```
when the **$** - variable contains **i** in its output it means the shell is interactive.
 
Let’s try out some fun filled bash script examples.

Create any file say myscript using your favorite editor like vi/vim write down the script as show below & set the executable permission to the file.

 
```bash
#!/bin/bash
echo -e "Hello World.....:)\nThis is my first bash script.."

```
OR
```bash
#!/bin/bash
echo "Hello World..... :)"
echo "This is my first bash script.."
```
Both the above script will print the message in two different lines.


The difference between the two is that in the first script both messages are used within the same echo command with additional options like “-e & \n”


**-e**         is used to enable interpretation of backslash escapes.


**\n**         is used to change the output to new line.
 

In this second example we will greet the user with a welcome message whenever he access the terminal.

 
Edit the **~/.bash_profile** using your favorite editor
```bash
#vi ~/.bash_profile

```
add the below lines to the end of the file
```bash

hname=`hostname`
usern=`whoami`
echo "Hello $usern welcome to $hname"

```
Now let’s understand what’s happing in the above script
 

The **~/.bash_profile** is executed when BASH is invoked as a login shell, certain environment variables are set here or can be set here for that particular user.


Adding the above lines will greet the user with those messages whenever he logins the terminal.


Example:
Username: admin
Hostname: testhost
 
The output after adding those lines will be
```bash
Hello admin welcome to testhost

```

