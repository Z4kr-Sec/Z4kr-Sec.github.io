---
title: "Easy"
date: 2025-04-27T23:09:18-04:00
draft: false
weight: 1
---
{{< figure
  src="/assets/images/HTB/easy.jpg"
  height= 200
  width= 400
>}}


## 2025


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

