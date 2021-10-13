# SEEDlabs: Shellshock Attack Lab

#### Ozgur Ural
#### Student ID: 2564455

## 1 Lab Description

On  September  24,  2014,  a  severe  vulnerability  in  Bash  was  identified.  Nicknamed  Shellshock,  this vulnerability can exploit many systems and be launched either remotely or from a local machine. In this  lab,  students  will  work  on  this  attack  to  understand  the  Shellshock  vulnerability.  The  learning objective  of  this  lab  is  for  students  to  get  a  hands-on  experience  on  this  interesting  attack, understand how it works, and reflect on the lessons learned from using this attack.


##  2 Lab Tasks

## 2.1 Environment Setup
The Bash program in Ubuntu 16.04 has already been patched, so it is no longer vulnerable to the  Shellshock  attack.  For  the  purpose  of  this  lab,  we  have  installed  a  vulnerable  version  of Bash inside the /bin folder; its name is bash shellshock. We need to use this Bash in our task. Please run this vulnerable version of Bash like the following and then design an experiment to verify whether this Bash is vulnerable to the Shellshock attack or not.

```sh
$ /bin/bash_shellshock
```

##  2.2    Task 1: Attack CGI programs
In  this  task,  we  will  launch  the  Shellshock  attack  on  a  remote  web  server.  Many  web  servers enable CGI, which is a standard method used to generate dynamic content on Web pages and Web  applications.  Many  CGI  programs  are  written  using  shell  script.  Therefore,  before  a  CGI program is executed, the shell program will be invoked first, and such an invocation is triggered by a user from a remote computer.

Set up the CGI Program. Create a very simple CGI program (called myprog.cgi) using the root account  in  folder  /usr/lib/cgi-bin/  as  the  following.  Make  this  cgi  executable  by  running  ”chmod 755 myprog.cgi”. It simply prints out "Hello, I am Shellshock!!" using shell script.

```sh
#!/bin/bash
echo "Content-type: text/plain"
echo
echo
echo "Hello, I am Shellshock!!"
```
