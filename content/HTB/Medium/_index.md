---
title: "Medium"
date: 2025-04-27T23:09:51-04:00
weight: 2
draft: false 
---
{{< figure
  src="/assets/images/HTB/medium.jpg"
  height= 200
  width= 400
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



Continue to **[Administrator](/HTB/medium/administrator/)**

##### Hands On!
{{< icon "HTB-icon" >}} Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/634)**

{{% /details %}}

## 2024

{{% details title="Survellance" closed="true" %}}


{{< figure
  src="/assets/images/HTB/survellance/Surveillance-LOGO.png"
  link="/HTB/medium/survellance-htb/"
  height= 400
  width= 400
>}}


Survellance is a medium machine of Hack The Box (HTB), the machine  begins with identifying a CMS vulnerability on the webpage hosted on port 80, which grants initial access to the system. Through enumeration, I uncovered a database file containing an encrypted password. Cracking this password allows me to access a ZoneMinder instance running on localhost. By exploiting a known vulnerability in ZoneMinder, I elevate my access to the 'zoneminder' user. The final step involves leveraging sudo privileges to achieve full root access.

Continue to **[Survellance](/HTB/medium/survellance-htb/)**

##### Hands On!
{{< icon "HTB-icon" >}} Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/580)**

{{% /details %}}