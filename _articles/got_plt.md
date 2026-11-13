---
title: "got plt"
date: 2026-11-13
summary: "got plt explanation"
---
readelf -rs

.plt: relocation id (0x0, 0x1.. etc)
.rela.plt: symbol table index
(000000003f70  000400000007 R_X86_64_JUMP_SLO 0000000000000000 mmap@GLIBC_2.2.5 + 0)
(offset        info         type              symbol value     name)

000400000007: 4: symbol table index, 7: R_X86_64_JUMP_SLO

0000000000001020 <.plt>:
    1020:       ff 35 32 2f 00 00      push   QWORD PTR [rip+0x2f32]        # 3f58 <_GLOBAL_OFFSET_TABLE_+0x8>
    1026:       ff 25 34 2f 00 00      jmp    QWORD PTR [rip+0x2f34]        # 3f60 <_GLOBAL_OFFSET_TABLE_+0x10>
    102c:       0f 1f 40 00            nop    DWORD PTR [rax+0x0]
    1030:       f3 0f 1e fa            endbr64
    1034:       68 00 00 00 00         push   0x0
    1039:       e9 e2 ff ff ff         jmp    1020 <_init+0x20>
    103e:       66 90                  xchg   ax,ax

relocation id in the .plt section (0x0) is the index in .rela.plt entry that it should use which contains
relocation id: 0x0, jumps to plt at address 0x1020, at 0x1020 pushes program id onto stack and jumps to 0x3f60, dynamic linker takes place looks at .rela.plt and resolves using symbol table index in .dynsym as it points to the section in >
(looks at libraries for symbol name to find offset in library then writes base of library + offset in library to GOT)

https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-di-frederico.pdf

GOT\[1] is the elf link map containing information about the elf program
GOT\[2] is _dl_resolve_runtime function _dl_resolve_runtime(link_map, relocation_index)
GOT\[3+] symbols to be resolved



