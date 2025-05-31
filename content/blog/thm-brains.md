+++
title = "TryHackMe Writeup — Brains"
date = "2025-05-19"
tags = ["ctf","security"]
+++

Room Link: [https://tryhackme.com/room/brains](https://tryhackme.com/room/brains)

### 1. Red: Exploit the Server!

`The city forgot to close its gate. Welcome to the Brains challenge, part of TryHackMe’s Hackathon! All brains gathered to build an engineering marvel; however, it seems strangers had found away to get in.`

![Image](/images/thm-brains/1.png)

As the **Ping** output shows, the TTL (Time to Live) value was 63. This is a strong indicator that the target machine was running a Linux distribution. With this information in hand, I proceeded to port scanning using **nmap** to discover open services.

![Image](/images/thm-brains/2.png)

The results revealed three open ports: SSH (port 22), HTTP (port 80), and another HTTP service on the **unusual port 50000**. The presence of a web server on a non-standard port immediately piqued my interest. Navigating to 50000 port in my browser, I was greeted with a login page.

![Image](/images/thm-brains/3.png)

A quick peek at the page source confirmed my suspicions – this was a **TeamCity CMS** instance.

![Image](/images/thm-brains/4.png)

With the CMS identified, the next logical step was to search for publicly known vulnerabilities. A quick search for "TeamCity exploits" led me to **CVE-2024-27198**, a critical authentication bypass vulnerability. It looked promising! I fired up Metasploit to leverage this exploit.

Metasploit successfully exploited the vulnerability, uploaded a plugin, and established a Meterpreter session! 

![Image](/images/thm-brains/5.png)

From the Meterpreter session, I quickly obtained a shell and escalated my privileges to root using `sudo -l` and `sudo su`. It seems the initial configuration was quite permissive, granting the ubuntu user passwordless sudo access.

### 2. Blue: Let's Investigate

`The Splunk instance will  be accessible at MACHINE_IP:8000 using the credentials mentioned below`

I. What is the name of the **backdoor user** which was created on the server after exploitation?

To answer this, I needed to look for events indicating the creation of a new user, specifically one with a shell assigned. My Splunk query was:

> host=brains "shell=/bin"

![Image](/images/thm-brains/6.png)

II. What is the name of the **malicious-looking package installed** on the server?

Knowing the approximate timeframe of the exploitation, I focused my search on package installations around that time. My Splunk query was:

> host=brains date_month=july date_hour=22 installed

This query filtered logs for events on the 'brains' host during July, specifically in the 22nd hour, looking for entries related to package installations.

![Image](/images/thm-brains/7.png)

III. What is the name of the **plugin installed** on the server **after successful exploitation**?

The Metasploit module we used likely uploaded a malicious plugin to achieve command execution. To find the name of this plugin, I searched for events related to plugin installations around the time of the exploit.

> host=brains date_month=july plugin

![Image](/images/thm-brains/8.png)

This query looked for logs on the 'brains' host in July that mentioned "plugin". The results revealed the name of the uploaded plugin.

--- 

I hope this writeup has been informative and engaging. Happy hacking!