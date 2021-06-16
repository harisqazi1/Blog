---
layout: default
title: Lian_Yu
grand_parent: Write-Ups
parent: TryHackMe
nav_order: 1
---

# Lian_Yu

### This is a room on the TryHackMe website at: [https://tryhackme.com/room/lianyu](https://tryhackme.com/room/lianyu)
--------------

I started off by doing an nmap scan:

```text
nmap -T4 -A -oN nmaptxt 10.10.121.53
```

I then got the following output:

```text
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-18 00:47 GMT
Nmap scan report for 10.10.121.53
Host is up (0.20s latency).
Not shown: 996 closed ports
PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 3.0.2
22/tcp open ssh OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey:
| 1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
| 2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
| 256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
| 256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp open http Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp open rpcbind 2-4 (RPC #100000)
| rpcinfo:
| program version port/proto service
| 100000 2,3,4 111/tcp rpcbind
| 100000 2,3,4 111/udp rpcbind
| 100000 3,4 111/tcp6 rpcbind
| 100000 3,4 111/udp6 rpcbind
| 100024 1 42412/tcp6 status
| 100024 1 52326/udp status
| 100024 1 52591/udp6 status
| 100024 1 58772/tcp status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

I see 4 ports open. My first idea is to open the main page.

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200106.png?raw=true)

Here, the word "arrow" was bolded, so I saved it, just in case it could mean something later on. I then ran a gobuster command, in order to enumerate the directories on the server. I ran the following command:

```text
gobuster dir -u http://10.10.121.53/ -w ../resources/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -t 40
```

In the previous command, the wordlist I am using comes from [this github page](https://github.com/danielmiessler/SecLists). I then found a directory called: "is----".

I then ran gobuster again with the addition of the new directory I had found:

```text
gobuster dir -u http://10.10.121.53/is----/ -w ../resources/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -t 40
```

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200208.png?raw=true)

I then found another directory called "2---". Looking at the source of this website, I found this:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200255.png?raw=true)

It is then shown that we need to find a file that has an extension with ".ticket". I then ran gobuster once more, but this time, I ran with an extension:

```text
gobuster dir -u http://10.10.121.53/is----/2---/ -w ../resources/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -t 40 -x .ticket
```

I then found a file with a .ticket extension.

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200336.png?raw=true)

What I had crossed out in red was a password of some sort. I then had to look at the clue on the TryHackMe Lian_Yu room. This led me to realize that the part crossed out in red was actually the ftp password in a base encoding.

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200424.png?raw=true)

I then went to CyberChef, and was able to crack the password:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200503.png?raw=true)

This gave me the ftp password. I then had to find the username. At this point, I did get stuck, and consulted [this write-up](https://infosecwriteups.com/tryhackme-lian-yu-ctf-writeup-detailed-7c229b1904fd) to find out what I missed. I realized that I had overlooked a word on a website. On the IP/is----/ website, I overlooked a word:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200549.png?raw=true)

This word was in white, and I was unable to see it initially since it matched the background. I then used "vigilante" as the username, and then used the password for the ftp I got earlier. I then got into ftp:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200635.png?raw=true)

Here I can see 4 files I need to get: The 2 PNGs, the one JPG, and the last file was the ".other user" file. I downloaded all of them using "mget *". This should download all the files to your local directory. Reading the ".other user", I came across this:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200723.png?raw=true)

Here I can see various names, which could be potential usernames and/or passwords. One main name was "Slade" was what stuck out to me. I got stuck here once again. I then looked at the [previous write-up](https://infosecwriteups.com/tryhackme-lian-yu-ctf-writeup-detailed-7c229b1904fd), in order to realize that I had to run "steghide" on the aa.jpg file. When you run steghide, we need a password to extract the data from the aa.jpg file. This is where the Leave-me-alone.png file comes into play. The header for this file is messed up, so we have to edit the header. I found the header online on Wikipedia:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200759.png?raw=true)

I then edited the header of the file using the "hexeditor":

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200845.png?raw=true)

I was then able to view the file:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615200941.png?raw=true)

This turned out to be the password for the "steghide" command. Running the following command:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615201030.png?raw=true)

Entering the password from the "Leave-me-alone.jpg", I then got the output file:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615201159.png?raw=true)

I then found the password for the ssh server for the user "slade". I then entered those credentials and then I got into the server:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615201241.png?raw=true)

Here my first idea was to run "ls" to find out files in my current directory. I then saw the flag in a "user.txt" file. I then had to upgrade my privileges to root. To do this I ran "sudo -l". This command lets you know what commands our user can do as the root user.

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615201319.png?raw=true)

Here it can be seen that the command I can run as root is "pkexec". I then went to [https://gtfobins.github.io/](https://gtfobins.github.io/) in order to find out how I can use this command to get to root. I then found this webpage:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615201356.png?raw=true)

Running this command, I got to root:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210615201427.png?raw=true)

Reading the root.txt file, I saw the last flag and was able to complete the box!

