---
layout: default
title: Relevant
grand_parent: Write-Ups
parent: TryHackMe
nav_order: 1
---

# Relevant

#### **This is a write-up for the room on TryHackMe located at: [https://tryhackme.com/room/relevant](https://tryhackme.com/room/relevant)**

### Nmap Scan:

I ran **nmap -T4 -A 10.10.161.100 -oN nmap\_output**, however this did not get me anywhere. It seemed to me that the host was blocking ping probes. I then ran **nmap -Pn 10.10.161.100**:

```c
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-03 15:25 EDT
Nmap scan report for 10.10.161.100
Host is up (0.23s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE
135/tcp  filtered msrpc
139/tcp  filtered netbios-ssn
3389/tcp filtered ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 120.16 seconds
```

I later ran **sudo nmap -p- -Pn -sS -A 10.10.161.100 -oN fullscan.txt** which got me way more information**:**

```c
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-03 15:37 EDT
Nmap scan report for 10.10.161.100
Host is up (0.22s latency).
Not shown: 65527 filtered ports
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 10.0
| http-methods: 
|\_  Potentially risky methods: TRACE
|\_http-server-header: Microsoft-IIS/10.0
|\_http-title: IIS Windows Server
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target\_Name: RELEVANT
|   NetBIOS\_Domain\_Name: RELEVANT
|   NetBIOS\_Computer\_Name: RELEVANT
|   DNS\_Domain\_Name: Relevant
|   DNS\_Computer\_Name: Relevant
|   Product\_Version: 10.0.14393
|\_  System\_Time: 2021-06-03T19:48:50+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2021-06-02T19:27:43
|\_Not valid after:  2021-12-02T19:27:43
|\_ssl-date: 2021-06-03T19:49:26+00:00; 0s from scanner time.
49663/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|\_  Potentially risky methods: TRACE
|\_http-server-header: Microsoft-IIS/10.0
|\_http-title: IIS Windows Server
49667/tcp open  msrpc              Microsoft Windows RPC
49669/tcp open  msrpc              Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2016|2012|2008 (91%)
OS CPE: cpe:/o:microsoft:windows\_server\_2016 cpe:/o:microsoft:windows\_server\_2012 cpe:/o:microsoft:windows\_server\_2008:r2
Aggressive OS guesses: Microsoft Windows Server 2016 (91%), Microsoft Windows Server 2012 (85%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|\_clock-skew: mean: 1h24m00s, deviation: 3h07m51s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\\x00
|   Workgroup: WORKGROUP\\x00
|\_  System time: 2021-06-03T12:48:49-07:00
| smb-security-mode: 
|   account\_used: guest
|   authentication\_level: user
|   challenge\_response: supported
|\_  message\_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|\_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-06-03T19:48:51
|\_  start\_date: 2021-06-03T19:28:19

TRACEROUTE (using port 139/tcp)
HOP RTT       ADDRESS
1   59.97 ms  10.2.0.1
2   ... 3
4   200.41 ms 10.10.161.100

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 722.09 seconds

```

### Foothold:

I needed to find a way into the system, or a foothold. I browsed to the IP address:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613181039.png?raw=true)

I wanted to see if SMB is being ran on this page, I then used metasploit to find out what version of SMB was running:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613194928.png?raw=true)

I then ran a nmap script just for smb "**nmap --script smb-os-discovery 10.10.161.100**":

```c
Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\\x00
|   Workgroup: WORKGROUP\\x00
|\_  System time: 2021-06-03T13:22:34-07:00
```

I believed that smb is my way in. I then ran **smbclient**:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613195354.png?raw=true)

I then noticed a unique Sharename **nt4wrksv**. I wanted to see what's in here. So I ran smbclient to have access to the share:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613195506.png?raw=true)

I then saw a **passwords.txt** file.

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613201216.png?raw=true)

I then wanted to download it:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613201351.png?raw=true)

I was able to then read the password file on my local system:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613201429.png?raw=true)

The passwords were encoded, and it seemed to be base64 to me. Using [https://www.base64decode.org/](https://www.base64decode.org/) I was able to decode the text:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613201605.png?raw=true)

To me, this seemed like a login for an SMB user. Viewing [this writeup](https://medium.com/cybersecpadawan/relevant-walk-through-on-tryhackme-f7dedfcb00dc), I noticed that the decoded had made me miss another password. I then had 2 logins:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613201833.png?raw=true)

I did not know what tool to use next to get further. I then used [the same write-up](https://medium.com/cybersecpadawan/relevant-walk-through-on-tryhackme-f7dedfcb00dc) and noticed that I would have to use **psexec** which is located in **/usr/share/doc/python3-impacket/examples**. I was then able to run it on the SMB:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613202057.png?raw=true)

The user for Bill did not work:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613202118.png?raw=true)

Now I wanted to move back and see another way inside. Both port 80 and 49663 both are running the same website:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613202205.png?raw=true)

I then ran **dirsearch** on the port 49663:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613202238.png?raw=true)

At this point we have access to a website which has the same contents as the SMB server:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613203226.png?raw=true)

### **Exploitation:**

In order to exploit this system, I will have to create a payload to then submit to the samba share. I will have to use msfvenom in order to create this. I uploaded a .exe and it did not work. I had to find another format to upload the file to. I then looked at the same aforementioned write-up to see what format they used. They used **.aspx**. I also did not know that Windows Server 2016 is a x64 architecture. With those in mind, I then made a payload:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613203401.png?raw=true)

I then had to upload this to the SMB server:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613204817.png?raw=true)

I then have to go to the web version on the payload so that way it can be initiated. Before I do that, I have to have a netcat listener open on another terminal. The payload worked:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613204853.png?raw=true)

Browsing around the file system I found the user flag:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613204926.png?raw=true)

This was my first Windows machine, so I had to refer to the write-up to know where to go next. I then learned that I would have to upload an exe called **PrintSpoofer** which impersonates privileges. I then downloaded it:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613205011.png?raw=true)

I then had to upload it. I did this using the same method as I did for the payload:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613205107.png?raw=true)

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613205116.png?raw=true)

I then ran:

```c
PrintSpoofer.exe -i -c cmd
```

which led me to this:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613205235.png?raw=true)

I then went to the Desktop of the Administrator and got the flag:

![](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613205303.png?raw=true)

### What I Learned:

I learned a lot from this machine. It was my first Windows machine so there was a little learning curve for me while attempting this machine. I learned about **whoami /priv** which was not something I had known about previously. Overall, a great Windows machine, however NOT recommended for Windows machine beginners.