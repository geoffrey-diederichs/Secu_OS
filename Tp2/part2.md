
# Part II : Observe

## Sommaire

- [Part II : Observe](#part-ii--observe)
  - [Sommaire](#sommaire)
  - [1. strace](#1-strace)
  - [2. sysdig](#2-sysdig)
    - [A. Intro](#a-intro)
    - [B. Use it](#b-use-it)
  - [3. Bonus : Stratoshark](#3-bonus--stratoshark)

## 1. strace

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `ls`**

```console
$ strace ls
```

```
write(1, "part2.md\nREADME.md\nTp2\n", 23) = 23
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `cat`**

```
$ strace cat part2.md
```

Ouverture du fichier :

```console
openat(AT_FDCWD, "part2.md", O_RDONLY)  = 3
```

Lecture du fichier : 

```
write(1, "\n# Part II : Observe\n\n## Sommair"..., 4071
```


🌞 **Utiliser `strace` pour tracer l'exécution de `curl example.org`**

```
$ strace -c curl google.com 1>/dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   219  100   219    0     0   3085      0 --:--:-- --:--:-- --:--:--  3128
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 28.28    0.002037          13       156           mmap
 27.82    0.002004           6       318           write
  6.72    0.000484          11        41         3 openat
  6.34    0.000457         457         1           execve
  5.82    0.000419          11        37           mprotect
  4.53    0.000326           7        45           close
  3.48    0.000251          19        13           poll
  3.32    0.000239           5        40           fstat
  3.30    0.000238           6        37           read
  1.90    0.000137          34         4         4 connect
  1.12    0.000081           8        10           setsockopt
  0.99    0.000071          14         5           socket
  0.97    0.000070          14         5           brk
  0.90    0.000065          65         1           sendto
  0.54    0.000039          13         3         2 recvfrom
  0.47    0.000034          34         1           clone3
  0.46    0.000033          33         1           munmap
  0.44    0.000032          10         3         2 ioctl
  0.37    0.000027           6         4           getsockname
  0.29    0.000021           4         5           rt_sigprocmask
  0.29    0.000021           7         3         1 newfstatat
  0.25    0.000018           4         4           fcntl
  0.18    0.000013           6         2           eventfd2
  0.17    0.000012           6         2           getrandom
  0.15    0.000011           5         2           pread64
  0.14    0.000010          10         1         1 access
  0.14    0.000010          10         1           getsockopt
  0.11    0.000008           4         2           rt_sigaction
  0.08    0.000006           6         1           prlimit64
  0.07    0.000005           5         1           lseek
  0.07    0.000005           5         1           arch_prctl
  0.07    0.000005           5         1           set_tid_address
  0.07    0.000005           5         1           set_robust_list
  0.07    0.000005           5         1           rseq
  0.06    0.000004           4         1           geteuid
------ ----------- ----------- --------- --------- ----------------
100.00    0.007203           9       754        13 total
```

## 2. sysdig

### A. Intro

### B. Use it

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

```console
$ sudo sysdig proc.name=ls
```

Depuis un autre terminal :

```console
$ ls
sysdig-0.39.0-x86_64.rpm
```

Écrire dans le terminal :

```console
41508 11:23:26.485132804 0 ls (3435.3435) < write res=41 data=.[0m.[01;31msysdig-0.39.0-x86_64.rpm.[0m.
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

```console
$ sudo sysdig proc.name=cat
```

Depuis un autre terminal :

```console
$ cat coucou.txt 
coucou
```

Ouvrir le fichier :

```console
1556 11:25:01.580378214 1 cat (3451.3451) > openat dirfd=-100(AT_FDCWD) name=coucou.txt(/home/rocky/coucou.txt) flags=1(O_RDONLY) mode=0
```

Écrire dans le terminal :

```console
1569 11:25:01.580572243 1 cat (3451.3451) < write res=7 data=coucou.
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

```console
$ sudo sysdig user.name=rocky
```

🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

[curl.scap](./curl.scap)
