# Part I : Learn

## Sommaire

- [Part I : Learn](#part-i--learn)
  - [Sommaire](#sommaire)
  - [1. Anatomy of a program](#1-anatomy-of-a-program)
    - [A. `file`](#a-file)
    - [B. `readelf`](#b-readelf)
    - [C. `ldd`](#c-ldd)
  - [2. Syscalls basics](#2-syscalls-basics)
    - [A. Syscall list](#a-syscall-list)
    - [B. `objdump`](#b-objdump)

## 1. Anatomy of a program

### A. `file`

🌞 **Utiliser `file` pour déterminer le type de :**

```console
$ file /bin/ls
/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=070bc6c817a91e4446dd99363b49ad5a491718e4, for GNU/Linux 3.2.0, stripped

$ file /bin/ip
/bin/ip: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=91f34f6ba5c4ecae4dcefe1878109262297fea19, for GNU/Linux 3.2.0, stripped

$ file ~/Downloads/sound_2018_10_11_14_14_37_128.mp3 
/home/geoffrey/Downloads/sound_2018_10_11_14_14_37_128.mp3: Audio file with ID3 version 2.4.0, contains: MPEG ADTS, layer III, v1, 64 kbps, 44.1 kHz, Stereo
```

`ls` et `ip` : des fichiers ELF exécutables en 64bits.

`sound_2018_10_11_14_14_37_128.mp3` : fichier audio avec des metadatas en format ID3.

### B. `readelf`

🌞 **Utiliser `readelf` sur le programme `ls`**

```console
$ readelf -h /bin/ip
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x10b50
  Start of program headers:          64 (bytes into file)
  Start of section headers:          715832 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         14
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30

$ readelf -S /bin/ip
There are 31 section headers, starting at offset 0xaec38:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .note.gnu.pr[...] NOTE             0000000000000350  00000350
       0000000000000020  0000000000000000   A       0     0     8
  [ 2] .note.gnu.bu[...] NOTE             0000000000000370  00000370
       0000000000000024  0000000000000000   A       0     0     4
  [ 3] .interp           PROGBITS         0000000000000394  00000394
       000000000000001c  0000000000000000   A       0     0     1
  [ 4] .gnu.hash         GNU_HASH         00000000000003b0  000003b0
       0000000000001260  0000000000000000   A       5     0     8
  [ 5] .dynsym           DYNSYM           0000000000001610  00001610
       00000000000043b0  0000000000000018   A       6     1     8
  [ 6] .dynstr           STRTAB           00000000000059c0  000059c0
       0000000000002821  0000000000000000   A       0     0     1
  [ 7] .gnu.version      VERSYM           00000000000081e2  000081e2
       00000000000005a4  0000000000000002   A       5     0     2
  [ 8] .gnu.version_r    VERNEED          0000000000008788  00008788
       0000000000000190  0000000000000000   A       6     5     8
  [ 9] .rela.dyn         RELA             0000000000008918  00008918
       0000000000004d10  0000000000000018   A       5     0     8
  [10] .rela.plt         RELA             000000000000d628  0000d628
       0000000000001230  0000000000000018  AI       5    25     8
  [11] .init             PROGBITS         000000000000f000  0000f000
       0000000000000017  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         000000000000f020  0000f020
       0000000000000c30  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         000000000000fc50  0000fc50
       0000000000000010  0000000000000008  AX       0     0     8
  [14] .text             PROGBITS         000000000000fc80  0000fc80
       000000000006f2d7  0000000000000000  AX       0     0     64
  [15] .fini             PROGBITS         000000000007ef58  0007ef58
       0000000000000009  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         000000000007f000  0007f000
       0000000000017d2c  0000000000000000   A       0     0     32
  [17] .eh_frame_hdr     PROGBITS         0000000000096d2c  00096d2c
       0000000000001d3c  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         0000000000098a68  00098a68
       000000000000c528  0000000000000000   A       0     0     8
  [19] .note.ABI-tag     NOTE             00000000000a4f90  000a4f90
       0000000000000020  0000000000000000   A       0     0     4
  [20] .note.package     NOTE             00000000000a4fb0  000a4fb0
       000000000000009c  0000000000000000   A       0     0     4
  [21] .init_array       INIT_ARRAY       00000000000a6c50  000a5c50
       0000000000000008  0000000000000008  WA       0     0     8
  [22] .fini_array       FINI_ARRAY       00000000000a6c58  000a5c58
       0000000000000008  0000000000000008  WA       0     0     8
  [23] .data.rel.ro      PROGBITS         00000000000a6c60  000a5c60
       0000000000001ad8  0000000000000000  WA       0     0     32
  [24] .dynamic          DYNAMIC          00000000000a8738  000a7738
       0000000000000240  0000000000000010  WA       6     0     8
  [25] .got              PROGBITS         00000000000a8978  000a7978
       0000000000000670  0000000000000008  WA       0     0     8
  [26] .data             PROGBITS         00000000000a9000  000a8000
       0000000000006a84  0000000000000000  WA       0     0     32
  [27] .bss              NOBITS           00000000000afaa0  000aea84
       000000000004cae0  0000000000000000  WA       0     0     32
  [28] .gnu_debugaltlink PROGBITS         0000000000000000  000aea84
       0000000000000048  0000000000000000           0     0     1
  [29] .gnu_debuglink    PROGBITS         0000000000000000  000aeacc
       0000000000000034  0000000000000000           0     0     4
  [30] .shstrtab         STRTAB           0000000000000000  000aeb00
       0000000000000134  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

Entrypoint : `0x10b50`.

### C. `ldd`

🌞 **Utiliser `ldd` sur le programme `ls`**

```console
$ ldd /bin/ls
	linux-vdso.so.1 (0x00007f828c48c000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f828c417000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f828c221000)
	libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x00007f828c172000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f828c48e000)
```

GLIBC : `libc.so.6`.

## 2. Syscalls basics

### A. Syscall list

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

Lire un fichier : `read` (0x00) pour lire depuis le file descriptor ouvert avec `open` (0x02).

Écrire dans un ficher : `write` (0x01) pour écrire sur le file descriptor ouvert avec `open` (0x02).

Exécuter un nouveau process : `execve` (0x3b) pour lancer nouveau process.

### B. `objdump`

🌞 **Utiliser `objdump`** sur la commande `ls`

```console
$ objdump -d /bin/ls --section=.text > ls_disa.txt
```

[ls_disa.txt](./ls_disa.txt)

Un call :

```
4700:	e8 8b f9 ff ff       	call   4090 <abort@plt>
```

🌞 **Utiliser `objdump`** sur la librairie Glibc

```
$ objdump -d /usr/lib/x86_64-linux-gnu/libc.so.6 |  grep syscall -B 4 | grep "0x3,%eax" -A 3
```

```
   9d4f0:	b8 03 00 00 00       	mov    $0x3,%eax
   9d4f5:	0f 05                	syscall
```
