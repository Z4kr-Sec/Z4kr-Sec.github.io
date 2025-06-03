---
title: "HTB - WriteUps."
date: 2025-04-26T16:19:56-04:00
toc: false
# Display the last modification date
displayUpdatedDate: true
enableInlineShortcodes: true
#use a wide wight
width: wide
#show the content of the HTB directory 
layout: list
## This will show all the files (Including Easy, Medium, Hard machines)
draft: false
---


{{< figure
  src="/assets/images/HTB/HTB-main-logo.png"
  
  height= 350
  width= 650
>}}

## 2025


{{% details title="Administrator" closed="true" %}}


{{< figure
  src="/assets/images/HTB/Administrator/Administrator-LOGO.png"
  link="/HTB/medium/administrator/"
  height= 400
  width= 400
>}}


This Hack The Box Administrator machine is focused on exploitation of Active Directory weaknesses. Starting with user credentials, we used Bloodhound to map the domain, quickly identifying exploitable `GenericAll` permissions that allowed us to reset passwords using tools like `net rpc`.

Our investigation then uncovered an FTP server with a PasswordSafe file. Cracking this file with `pwsafe2john` and John the Ripper provided further credentials, which we leveraged to exploit ForceChangePassword privileges and perform a targeted Kerberoast for even higher access. The final step involved a `DCSync` attack via Impacket, retrieving domain admin hashes and granting complete control. This showcases a direct path from initial access to domain dominance by chaining AD misconfigurations and service vulnerabilities.



Continue to **[Administartor](/HTB/medium/administrator/)**

##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/634)**

{{% /details %}}


{{% details title="Sea" closed="true" %}}


{{< figure
  src="/assets/images/HTB/Sea/sea%20LOGO.png"
  link="/htb/easy/sea-htb/"
  height= 400
  width= 400
>}}


Sea is an Easy-rated Linux machine on Hack The Box that requires thorough web enumeration to uncover hidden directories and identify a vulnerable theme. Exploiting CVE-2023-41425 allows for remote code execution, leading to an initial foothold. A hashed password found in a database file is cracked to gain SSH access as a user. Privilege escalation is achieved by tunneling into a locally hosted service, leveraging access logs to execute commands as root.

Continue to **[Sea](/htb/easy/sea-htb/)**

##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/620)**

{{% /details %}}



## 2024

{{% details title="Survellance" closed="true" %}}


{{< figure
  src="/assets/images/HTB/survellance/Surveillance-LOGO.png"
  link="/htb/medium/survellance-htb/"
  height= 400
  width= 400
>}}


Survellance is a medium machine of Hack The Box (HTB), the machine  begins with identifying a CMS vulnerability on the webpage hosted on port 80, which grants initial access to the system. Through enumeration, I uncovered a database file containing an encrypted password. Cracking this password allows me to access a ZoneMinder instance running on localhost. By exploiting a known vulnerability in ZoneMinder, I elevate my access to the 'zoneminder' user. The final step involves leveraging sudo privileges to achieve full root access.

Continue to **[Survellance](/htb/medium/survellance-htb/)**

##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/580)**

{{% /details %}}



{{% details title="Analytics" closed="true"%}}

{{< figure
  src="/assets/images/HTB/Analytics/Analitics-LOGO.png"
  link="/htb/easy/analytics-htb/"
  height= 400
  width= 400
>}}



This machine focuses on exploiting a vulnerable instance of *Metabase* to gain initial access. with the help of the exploit - CVE-2023-38646, we obtain command execution on the target. The next phase involves navigating a Docker container environment, leveraging exposed credentials, and transitioning to an SSH session with user-level access. Finally, the privilege escalation is achieved by exploiting a known vulnerability (CVE-2023-2640) in the operating system to gain root access.

Continue to **[Analytics](/htb/easy/analytics-htb/)**

##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/569)**

{{% /details %}}





{{% details title="Delivery" closed="true" %}}

CREATE AND INSERT EXCERPT FOR DELIVERY MACCHINE

Continue to **[Delivery](/htb/easy/delivery-htb/)**
##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/308)**

{{% /details %}}

