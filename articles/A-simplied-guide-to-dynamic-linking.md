# A Simplied Guide To Dynamic Linking

A demo program that you already know:

```
#include <stdio.h>

int main()
{
  printf("Hello World\n");
  return 0;
}
```

`nm` is a tool that displays the symbol table associated with an object, archive library of objects, or executable file. Let's fire `nm` to see what we have:

```
$ gcc hello.c -m32
$ file ./a.out
./a.out: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=e3ffe81f21e3c81aa531aa203fcd714f5e5d2de0, not stripped
$ nm -a ./a.out
00000000 a
080496cc b .bss # uninitialized data section
080496cc B __bss_start
00000000 n .comment
080496cc b completed.6862
00000000 a crtstuff.c
00000000 a crtstuff.c
080496c4 d .data # initialized data section
080496c4 D __data_start
080496c4 W data_start
08048340 t deregister_tm_clones
080483b0 t __do_global_dtors_aux
080495b8 t __do_global_dtors_aux_fini_array_entry
080496c8 D __dso_handle
080495c0 d .dynamic
080495c0 d _DYNAMIC
080481fc r .dynstr
080481ac r .dynsym
080496cc D _edata
080484e8 r .eh_frame
080484bc r .eh_frame_hdr
080496d0 B _end
08048494 T _fini
08048494 t .fini
080495b8 t .fini_array
080484a8 R _fp_hw
080483d0 t frame_dummy
080495b4 t __frame_dummy_init_array_entry
080485b0 r __FRAME_END__
080496ac d _GLOBAL_OFFSET_TABLE_
         w __gmon_start__
0804818c r .gnu.hash
08048246 r .gnu.version
08048250 r .gnu.version_r
080496a8 d .got
080496ac d .got.plt
00000000 a hello.c
08048290 T _init
08048290 t .init
080495b4 t .init_array
080495b8 t __init_array_end
080495b4 t __init_array_start
00000000 a init.c
08048134 r .interp # read only data section
080484ac R _IO_stdin_used
         w _ITM_deregisterTMCloneTable
         w _ITM_registerTMCloneTable
080495bc d .jcr
080495bc d __JCR_END__
080495bc d __JCR_LIST__
         w _Jv_RegisterClasses
08048490 T __libc_csu_fini
08048430 T __libc_csu_init
         U __libc_start_main@@GLIBC_2.0
080483fb T main
08048148 r .note.ABI-tag
08048168 r .note.gnu.build-id
080482c0 t .plt
         U puts@@GLIBC_2.0 # our printf in the source code
08048370 t register_tm_clones
08048270 r .rel.dyn
08048278 r .rel.plt
080484a8 r .rodata
08048300 T _start
08048300 t .text
080496cc D __TMC_END__
08048330 T __x86.get_pc_thunk.bx
```

The second column stands for the type of the section. I need to carefully look at the .got section.

```
$ readelf -r ./a.out

Relocation section '.rel.dyn' at offset 0x270 contains 1 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
080496a8  00000206 R_386_GLOB_DAT    00000000   __gmon_start__

Relocation section '.rel.plt' at offset 0x278 contains 3 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
080496b8  00000107 R_386_JUMP_SLOT   00000000   puts
080496bc  00000207 R_386_JUMP_SLOT   00000000   __gmon_start__
080496c0  00000307 R_386_JUMP_SLOT   00000000   __libc_start_main
```

```
gdb-peda$ disas main
Dump of assembler code for function main:
   0x080483fb <+0>: lea    ecx,[esp+0x4]
   0x080483ff <+4>: and    esp,0xfffffff0
   0x08048402 <+7>: push   DWORD PTR [ecx-0x4]
   0x08048405 <+10>:  push   ebp
   0x08048406 <+11>:  mov    ebp,esp
   0x08048408 <+13>:  push   ecx
   0x08048409 <+14>:  sub    esp,0x4
   0x0804840c <+17>:  sub    esp,0xc
   0x0804840f <+20>:  push   0x80484b0
   0x08048414 <+25>:  call   0x80482d0 <puts@plt>
   0x08048419 <+30>:  add    esp,0x10
   0x0804841c <+33>:  mov    eax,0x0
   0x08048421 <+38>:  mov    ecx,DWORD PTR [ebp-0x4]
   0x08048424 <+41>:  leave
   0x08048425 <+42>:  lea    esp,[ecx-0x4]
   0x08048428 <+45>:  ret
End of assembler dump.
gdb-peda$ disas 0x80482d0
Dump of assembler code for function puts@plt:
   0x080482d0 <+0>: jmp    DWORD PTR ds:0x80496b8
   0x080482d6 <+6>: push   0x0
   0x080482db <+11>:  jmp    0x80482c0
End of assembler dump.
```

Since the `ld` does dynamic linking, I set a breakpoint and follow the execution. If you place a endless while loop around `puts`, you will see the execution always jumps to `puts@plt`. That's the `plt` entry of `puts`. But how lazy binding is really done each time is subtly different.

```
Contents of section .got.plt:
 80496ac c0950408 00000000 00000000 d6820408  ................
 80496bc e6820408 f6820408
```

 Above is the original content of `.got.plt` from `objdump`. Let me confirm this in `gdb`:

```
gdb-peda$ x/x 0x80496b8
0x80496b8 <puts@got.plt>: 0x080482d6
```

Since my system is IA32, you must reverse the content. This reveals d6820408. This address happens to be the instruction right behind `jmp    DWORD PTR ds:0x80496b8`. Continue to `stepi` to `0x80482c0`.

```
   0xf7fee8c1 <_dl_runtime_resolve+1>:  push   ecx
   0xf7fee8c2 <_dl_runtime_resolve+2>:  push   edx
   0xf7fee8c3 <_dl_runtime_resolve+3>:  mov    edx,DWORD PTR [esp+0x10]
=> 0xf7fee8c7 <_dl_runtime_resolve+7>:  mov    eax,DWORD PTR [esp+0xc]
   0xf7fee8cb <_dl_runtime_resolve+11>: call   0xf7fe80e0 <_dl_fixup>
   0xf7fee8d0 <_dl_runtime_resolve+16>: pop    edx
   0xf7fee8d1 <_dl_runtime_resolve+17>: mov    ecx,DWORD PTR [esp]
   0xf7fee8d4 <_dl_runtime_resolve+20>: mov    DWORD PTR [esp],eax
```

`_dl_fixup` is the one who is responsible for fixing the `.got.plt` entry of `puts`.

```
gdb-peda$ b *0xf7fee8d0
Breakpoint 3 at 0xf7fee8d0
gdb-peda$ c
Continuing.
<snip>
gdb-peda$ x/x 0x80496b8
0x80496b8 <puts@got.plt>: 0xf7e655b0
gdb-peda$ disas 0xf7e655b0
Dump of assembler code for function puts:
   0xf7e655b0 <+0>: push   ebp
   0xf7e655b1 <+1>: mov    ebp,esp
   0xf7e655b3 <+3>: push   edi
   0xf7e655b4 <+4>: push   esi
   0xf7e655b5 <+5>: push   ebx
   0xf7e655b6 <+6>: call   0xf7f24575 <__x86.get_pc_thunk.bx>
   0xf7e655bb <+11>:  add    ebx,0x150a45
   0xf7e655c1 <+17>:  sub    esp,0x28
   0xf7e655c4 <+20>:  push   DWORD PTR [ebp+0x8]
```

The entry is finally fixed. Next time when `puts` is called, the execution flow will directly jump to the `puts` code.

