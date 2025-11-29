# RootMe

---

link: <https://tryhackme.com/room/rrootme>

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

![img1](assets/Pasted%20image%2020250711160201.png)

We see that 2 ports (22/ssh and 80/http) are open on the target machine. Port 22 is running ssh and the Apache version is `2.4.29`

## gobuster
Since port 80 is open we can visit the website hosted on it:

![img2](assets/Screenshot%20from%202025-07-11%2013-50-44%201.png)


Next we use `gobuster` to perform file enumeration to discover hidden directories and files on the web server

`gobuster dir -u <target_url> -w <wordlist>`

![img3](assets/Pasted%20image%2020250711162654.png)

We see 2 interesting directories `/uploads` and `/panel` 
visiting both:

- `http://<target_IP>/panel` $\rightarrow$ leads us to a page containing an upload form

![img4](assets/Screenshot%20from%202025-07-11%2013-56-28.png)



- `http://<target_IP>/uploads` $\rightarrow$ the directory where we can potentially see the files uploaded  

![img5](assets/Screenshot%20from%202025-07-11%2013-56-54.png)


# Getting a shell

Since our website contains an upload form, we can use the [php-reverse-shell payload](https://github.com/pentestmonkey/php-reverse-shell) . The script will open an outbound TCP connection from the webserver to a host and port of your choice.  Bound to this TCP connection will be a shell.

Download the `php-reverse-shell.php` file from github and configure the file to connect to your machine.

This can be done by changing the IP in the file to your attacking machine’s IP.
We can also choose a different port to listen on.`

![img6](assets/Screenshot%20from%202025-07-11%2014-02-39.png)

After configuring the file save and upload it

![img7](assets/Screenshot%20from%202025-07-11%2014-04-06.png)

![img8](assets/Screenshot%20from%202025-07-11%2014-03-52.png)

This gives an error which roughly translates to 'uploading php files is not allowed'
One of the easiest ways to bypass file uploading filtering is to rename it. We can rename the file extension to older `php` formats like 
- `.phtml`
- `.php3` 
- `.php4`
- `.php5` 
- `.inc`
etc...

![img9](assets/Screenshot%20from%202025-07-11%2014-04-36.png)

uploading this renamed file:

![img10](assets/Screenshot%20from%202025-07-11%2014-05-01.png)

Success !!!

We can find our uploaded file in the `/uploads` directory

![img11](assets/Screenshot%20from%202025-07-11%2014-05-19.png)

Before executing this we need to open a `netcat` listener 

![img12](assets/Screenshot%20from%202025-07-11%2014-05-42.png)

Once the listener is listening, execute the uploaded file and we will receive a connection on our terminal.

![img13](assets/Screenshot%20from%202025-07-11%2014-06-00.png)

We can upgrade the shell using the `pty` python library

`python -c 'import pty; pty.spawn("/bin/bash")'`

![img14](assets/Screenshot%20from%202025-07-11%2014-13-42.png)

Use `find` to locate the `user.txt` file

![img15](assets/Screenshot%20from%202025-07-11%2014-14-47.png)

`cat` the file to obtain the flag

![img16](assets/Pasted%20image%2020250711192121.png)


# Privilege escalation

The first task under privilege escalation is a big hint. We need to search for a file with SUID permission

> [!NOTE]
> this is a special permission in the linux filesystem 
> assigned to executable files, where it allows users who execute the file to temporarily assume the privileges of the file's owner $\implies$ a user without the necessary permission can access/execute these files.

We can use `find` to locate all files that have the SUID permission

![img17](assets/Pasted%20image%2020250726234505.png)

From the list of files, we can exploit the python file by using [GTFObins](https://gtfobins.github.io/)

Search GTFObins on how to exploit SUID permissions in python files:

![img18](assets/Screenshot%20from%202025-07-11%2014-17-29.png)
![img19](assets/Screenshot%20from%202025-07-11%2014-17-49.png)

Use the command to escalate privilege to `root`

![img20](assets/Screenshot%20from%202025-07-11%2014-21-24.png)

Find the `root.txt` file:

![img21](assets/Screenshot%20from%202025-07-11%2014-22-59.png)

`cat` the `root.txt` file to capture the last flag:
![img22](assets/Pasted%20image%2020250726235717.png)
