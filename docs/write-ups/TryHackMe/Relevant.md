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

![alt text](https://github.com/harisqazi1/blog/blob/main/assets/Pasted%20image%2020210613181039.png?raw=true)