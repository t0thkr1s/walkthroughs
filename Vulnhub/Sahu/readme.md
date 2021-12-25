# Sahu

## Port Scanning

```
t0thkr1s@kali ~> nmap -A -Pn -p- 192.168.1.73
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-17 06:30 CDT
Nmap scan report for 192.168.1.73
Host is up (0.0012s latency).
Not shown: 65530 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             230 Jan 30 13:55 ftp.zip
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
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)                                                           
| ssh-hostkey: 
|   3072 e2:78:c5:73:f2:86:cb:cb:02:7f:b6:72:85:61:ac:91 (RSA)
|   256 22:1a:ee:1a:98:4f:32:e7:dc:30:43:52:2c:b2:24:06 (ECDSA)
|_  256 1a:9b:28:b3:ad:58:32:e9:6c:f3:ea:3b:cf:6b:08:ad (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title.
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAHU)
445/tcp open  netbios-ssn Samba smbd 4.10.7-Ubuntu (workgroup: SAHU)
Service Info: Host: SAHU-VIRTUALBOX; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## FTP Enumeration

```
t0thkr1s@kali ~> ftp 192.168.1.73
Connected to 192.168.1.73.
220 (vsFTPd 3.0.3)
Name (192.168.1.73:t0thkr1s): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        133          4096 Jan 30 14:06 .
drwxr-xr-x    2 0        133          4096 Jan 30 14:06 ..
-rw-r--r--    1 0        0             230 Jan 30 13:55 ftp.zip
226 Directory send OK.
ftp> get ftp.zip
local: ftp.zip remote: ftp.zip
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ftp.zip (230 bytes).
226 Transfer complete.
230 bytes received in 0.00 secs (78.6723 kB/s)
ftp> exit
221 Goodbye.
t0thkr1s@kali ~> unzip ftp.zip 
Archive:  ftp.zip
[ftp.zip] ftp.txt password: 
   skipping: ftp.txt                 incorrect password
t0thkr1s@kali ~ [82]>
```

## HTTP Enumeration

```
t0thkr1s@kali ~> dirb http://192.168.1.73

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Mar 17 06:47:02 2020
URL_BASE: http://192.168.1.73/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.73/ ----
==> DIRECTORY: http://192.168.1.73/H/                                                                                                        
+ http://192.168.1.73/index.php (CODE:200|SIZE:194)                                                                                          
+ http://192.168.1.73/server-status (CODE:403|SIZE:277)                                                                                      
                                                                                                                                             
---- Entering directory: http://192.168.1.73/H/ ----
==> DIRECTORY: http://192.168.1.73/H/A/                                                                                                      
                                                                                                                                             
---- Entering directory: http://192.168.1.73/H/A/ ----
==> DIRECTORY: http://192.168.1.73/H/A/R/                                                                                                    
                                                                                                                                             
---- Entering directory: http://192.168.1.73/H/A/R/ ----
                                                                                                                                             
-----------------
END_TIME: Tue Mar 17 06:47:16 2020
DOWNLOADED: 18448 - FOUND: 2
```

