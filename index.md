###Nest HTB-Machine

##Summary
Nest is a easy windows machine with ip ```10.10.10.178```



##Enumeration-Nmap
```
root@strike:~# nmap -sC -sV 10.10.10.178
Starting Nmap 7.70 ( https://nmap.org ) at 2020-05-27 16:51 EET
Nmap scan report for 10.10.10.178
Host is up (0.073s latency).
Not shown: 999 filtered ports
PORT    STATE SERVICE       VERSION
445/tcp open  microsoft-ds?

Host script results:
|_clock-skew: mean: 4m59s, deviation: 0s, median: 4m59s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-05-27 16:57:14
|_  start_date: 2020-05-27 15:09:41

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.08 seconds
root@strike:~#
```
