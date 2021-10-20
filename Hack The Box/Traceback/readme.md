# Traceback

> OS: Linux Difficulty: Easy Points: 20 Release: 14 Mar 2020 IP: 10.10.10.181

## Port Scanning

```
t0thkr1s@kali ~> nmap -A -Pn -p- 10.10.10.181
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-15 00:37 CDT
Nmap scan report for 10.10.10.181
Host is up (0.047s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

WebShell Link: https://github.com/Xh4H/Web-Shells/blob/master/smevk.php

