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