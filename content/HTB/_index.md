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
---


{{< figure
  src="/assets/images/HTB/HTB-main-logo.png"
  
  height= 350
  width= 650
>}}

## 2025


{{% details title="Sea" closed="true" %}}


{{< figure
  src="/assets/images/HTB/Sea/sea%20LOGO.png"
  link="/HTB/easy/sea-htb/"
  height= 400
  width= 400
>}}


Sea is an Easy-rated Linux machine on Hack The Box that requires thorough web enumeration to uncover hidden directories and identify a vulnerable theme. Exploiting CVE-2023-41425 allows for remote code execution, leading to an initial foothold. A hashed password found in a database file is cracked to gain SSH access as a user. Privilege escalation is achieved by tunneling into a locally hosted service, leveraging access logs to execute commands as root.

Continue to **[Sea](/HTB/easy/sea-htb/)**

##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/620)**

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
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/580)**

{{% /details %}}



{{% details title="Analytics" closed="true"%}}

{{< figure
  src="/assets/images/HTB/Analytics/Analitics-LOGO.png"
  link="/HTB/easy/analytics-htb/"
  height= 400
  width= 400
>}}



This machine focuses on exploiting a vulnerable instance of *Metabase* to gain initial access. with the help of the exploit - CVE-2023-38646, we obtain command execution on the target. The next phase involves navigating a Docker container environment, leveraging exposed credentials, and transitioning to an SSH session with user-level access. Finally, the privilege escalation is achieved by exploiting a known vulnerability (CVE-2023-2640) in the operating system to gain root access.

Continue to **[Analytics](/HTB/easy/analytics-htb/)**

##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/569)**

{{% /details %}}





{{% details title="Delivery" closed="true" %}}

CREATE AND INSERT EXCERPT FOR DELIVERY MACCHINE

Continue to **[Delivery](/HTB/easy/delivery-htb/)**
##### Hands On!
Turn on the machine on **[Hack The Box.](https://app.hackthebox.com/machines/308)**

{{% /details %}}

