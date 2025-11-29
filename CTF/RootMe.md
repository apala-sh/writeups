---
link: https://tryhackme.com/room/rrootme
tags:
  - easy
  - thm
  - ctf
---



# Deploy the machine
Connect to TryHackMe network and deploy the target machine. 

# Reconnaissance

## namp scan
scan the target machine using `nmap` to find all the open ports and the service/version running on said ports
`namp -sSCV -oN nmapScan <target_IP>`
- `-sS` $\implies$ TCP SYN scan
- `-sV` $\implies$ Probe open ports to determine service/version info
- `-sC` $\implies$ use default script (or `--script=default`)

![[Pasted image 20250711160201.png]]

We see that 2 ports (22/ssh and 80/http) are open on the target machine. Port 22 is running ssh and the Apache version is `2.4.29`

## gobuster
Since port 80 is open we can visit the website hosted on it:

![[Screenshot from 2025-07-11 13-50-44 1.png]]


Next we use `gobuster` to perform file enumeration to discover hidden directories and files on the web server

`gobuster dir -u <target_url> -w <wordlist>`

![[Pasted image 20250711162654.png]]

We see 2 interesting directories `/uploads` and `/panel` 
visiting both:

- `http://<target_IP>/panel` $\rightarrow$ leads us to a page containing an upload form

![[Screenshot from 2025-07-11 13-56-28.png]]



- `http://<target_IP>/uploads` $\rightarrow$ the directory where we can potentially see the files uploaded  

![[Screenshot from 2025-07-11 13-56-54.png]]


# Getting a shell

Since our website contains an upload form, we can use the [php-reverse-shell payload](https://github.com/pentestmonkey/php-reverse-shell) . The script will open an outbound TCP connection from the webserver to a host and port of your choice.  Bound to this TCP connection will be a shell.

Download the `php-reverse-shell.php` file from github and configure the file to connect to your machine.

This can be done by changing the IP in the file to your attacking machine’s IP.
We can also choose a different port to listen on.`

![[Screenshot from 2025-07-11 14-02-39.png]]
After configuring the file save and upload it

![[Screenshot from 2025-07-11 14-04-06.png]]

![[Screenshot from 2025-07-11 14-03-52.png]]

This gives an error which roughly translates to 'uploading php files is not allowed'
One of the easiest ways to bypass file uploading filtering is to rename it. We can rename the file extension to older `php` formats like 
- `.phtml`
- `.php3` 
- `.php4`
- `.php5` 
- `.inc`
etc...

![[Screenshot from 2025-07-11 14-04-36.png]]
uploading this renamed file:

![[Screenshot from 2025-07-11 14-05-01.png]]

Success !!!

We can find our uploaded file in the `/uploads` directory

![[Screenshot from 2025-07-11 14-05-19.png]]

Before executing this we need to open a `netcat` listener 

![[Screenshot from 2025-07-11 14-05-42.png]]

Once the listener is listening, execute the uploaded file and we will receive a connection on our terminal.

![[Screenshot from 2025-07-11 14-06-00.png]]

We can upgrade the shell using the `pty` python library

`python -c 'import pty; pty.spawn("/bin/bash")'`

![[Screenshot from 2025-07-11 14-13-42.png]]

Use `find` to locate the `user.txt` file

![[Screenshot from 2025-07-11 14-14-47.png]]
`cat` the file to obtain the flag
![[Pasted image 20250711192121.png]]


# Privilege escalation

The first task under privilege escalation is a big hint. We need to search for a file with [[Linux permissions#SUID $ implies$ user + s (pecial)|SUID permission]]

> [!NOTE]
> this is a special permission in the linux filesystem 
> assigned to executable files, where it allows users who execute the file to temporarily assume the privileges of the file's owner $\implies$ a user without the necessary permission can access/execute these files.

We can use `find` to locate all files that have the SUID permission
![[Pasted image 20250726234505.png]]
From the list of files, we can exploit the python file by using [GTFObins](https://gtfobins.github.io/)

Search GTFObins on how to exploit SUID permissions in python files:

![[Screenshot from 2025-07-11 14-17-29.png]]![[Screenshot from 2025-07-11 14-17-49.png]]

Use the command to escalate privilege to `root`

![[Screenshot from 2025-07-11 14-21-24.png]]

Find the `root.txt` file:

![[Screenshot from 2025-07-11 14-22-59.png]]

`cat` the `root.txt` file to capture the last flag:
![[Pasted image 20250726235717.png]]
