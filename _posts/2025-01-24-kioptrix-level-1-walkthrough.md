---
title: "Kioptrix: Level 1 Walkthrough"
date: 2025-01-24T00:00:00+00:00
category: Walkthrough
tags: ["walkthrough", "oscp", "ctf", "kioptrix"]
description: "Step-by-step guide to solving the Kioptrix Level 1 machine, covering setup, enumeration, multiple exploitation methods, and privilege escalation. Ideal for beginners."
toc: true
comments: true
---

# **Kioptrix: Level 1 Walkthrough**

---

## **Introduction**

The **Kioptrix Level 1** machine is a beginner-friendly challenge designed to teach core penetration testing skills, including:

- **Network and service enumeration**
- **Exploitation of common vulnerabilities**
- **Privilege escalation techniques**

This walkthrough assumes you have a basic understanding of penetration testing tools and methodologies. Let’s dive in!

---

## **Setup and Configuration**

Before starting the exploitation process, ensure the virtual machine is set up correctly in **VMware Workstation** or **VMware Player**.

### **Steps to Set Up Kioptrix Level 1**

1. **Download** the VM image from the official source.
2. Import the VM into VMware Workstation or Player.
3. Allocate a minimum of **512MB RAM** for smooth operation.
4. Configure the **network adapter** to **NAT Mode** for proper connectivity.

---

### **Common VMware Network Issue**

One common issue faced during the setup is the **network adapter reverting to Bridged Mode**, even when explicitly set to NAT Mode. This can disrupt communication between the host and the virtual machine.

#### **Solution**

To resolve this issue:

1. Locate the VMware configuration file for the VM (e.g., `Kioptrix Level 1.vmx`).
2. Open the file in a text editor and search for the line:
3. Edit its value to `NAT` as shown below.

   <div style="background-color: #f4f4f4; border: 1px solid #ccc; padding: 10px; font-family: 'Courier New', Courier, monospace;">
   memsize = "64"<br>
   ide1:0.present = "FALSE"<br>
   ide1:0.fileName = "F:"<br>
   ide1:0.deviceType = "atapi-cdrom"<br>
   ide1:0.allowGuestConnectionControl = "FALSE"<br>
   ide1:1.present = "FALSE"<br>
   ide1:1.fileName = "Kioptix Level 1.vmdk"<br>
   ide1:1.writeThrough = "TRUE"<br>
   ethernet0.present = "TRUE"<br>
   ethernet0.allowGuestConnectionControl = "FALSE"<br>
   ethernet0.features = "1"<br>
   ethernet0.wakeOnPcktRcv = "FALSE"<br>
   <span style="background-color: yellow; color: black;">ethernet0.networkName = "NAT"</span><br>
   ethernet0.addressType = "generated"<br>
   guestOS = "other24xlinux"<br>
   uuid.location = "56 4d 0e e7 c2 81 21 e5-2d e6 61 b1 79 11 3d da"<br>
   uuid.bios = "56 4d 0e e7 c2 81 21 e5-2d e6 61 b1 79 11 3d da"<br>
   vc.uuid = "52 77 3c 2e 12 81 3a 68-25 23 b3 92 4e 8e 01 ff"<br>
   </div>

4. Now save the configuration file and restart the VM.

---
## **Enumeration**

### **Identifying the Target Machine**

The first step in enumeration is to identify the IP address of the **Kioptrix Level 1** machine in the local network. We can achieve this using the tool **netdiscover**, which is pre-installed in most penetration testing distributions like **Kali Linux**.

#### **Steps to Identify the Target**

1. Open a terminal on your attacker machine.
2. Run the following command to discover all active devices in the network:
   ```bash
   sudo netdiscover -r <NETWORK_RANGE>
   ```

    ```bash
    Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                   
 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240                                                                                            
 _____________________________________________________________________________
    IP               At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.153.1   00:50:56:c0:00:08      1      60  VMware, Inc.                                                                                             
 192.168.153.2   00:50:56:e5:33:16      1      60  VMware, Inc.                                                                                             
 192.168.153.129 00:0c:29:11:3d:da      1      60  VMware, Inc.                                                                                             
 192.168.153.254 00:50:56:ee:1f:12      1      60  VMware, Inc.
 ```
3. The IP of the target host is 192.168.153.129

---

## **Service Enumeration**

Once we have identified the target machine's IP address, the next step is to perform service enumeration to discover running services and their versions.

### **Port Scanning with Nmap**

We will use **Nmap** to identify open ports and running services on the target machine.

1. Run the following command to scan the target for open ports and service versions:

   ```bash
   sudo nmap -p- -sC -sV -O 192.168.153.129 --open
   ```
```bash
Nmap scan report for 192.168.153.129
Host is up (0.00088s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
| http-methods: 
|_  Potentially risky methods: TRACE
111/tcp  open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1           1024/tcp   status
|_  100024  1           1024/udp   status
139/tcp  open  netbios-ssn Samba smbd (workgroup: IMYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_ssl-date: 2025-01-27T05:00:04+00:00; +1h01m47s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
|_http-title: 400 Bad Request
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after:  2010-09-26T09:32:06
1024/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:11:3D:DA (VMware)
Device type: general purpose|media device
Running: Linux 2.4.X, Roku embedded
OS CPE: cpe:/o:linux:linux_kernel:2.4 cpe:/h:roku:soundbridge_m1500
OS details: Linux 2.4.9 - 2.4.18 (likely embedded), Roku HD1500 media player
Network Distance: 1 hop 
----------------------------------------------------------------
Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: 1h01m46s
```
## **Web Enumeration**
### **Nikto scan**
```bash
┌──(root㉿KALI)-[/home/hrishi]
└─# nikto -host http://192.168.153.129/       
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.153.129
+ Target Hostname:    192.168.153.129
+ Target Port:        80
+ Start Time:         2025-01-27 09:44:58 (GMT5.5)
---------------------------------------------------------------------------
+ Server: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
+ /: Server may leak inodes via ETags, header found with file /, inode: 34821, size: 2890, mtime: Thu Sep  6 08:42:46 2001. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /: Apache is vulnerable to XSS via the Expect header. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-3918
+ Apache/1.3.20 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ mod_ssl/2.8.4 appears to be outdated (current is at least 2.9.6) (may depend on server version).
+ OpenSSL/0.9.6b appears to be outdated (current is at least 3.0.7). OpenSSL 1.1.1s is current for the 1.x branch and will be supported until Nov 11 2023.
+ Apache/1.3.20 - Apache 1.x up 1.2.34 are vulnerable to a remote DoS and possible code execution.
+ Apache/1.3.20 - Apache 1.3 below 1.3.27 are vulnerable to a local buffer overflow which allows attackers to kill any process on the system.
+ Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi.
+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell.
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE .
+ /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
+ ///etc/hosts: The server install allows reading of any system file by adding an extra '/' to the URL.
+ /usage/: Webalizer may be installed. Versions lower than 2.01-09 vulnerable to Cross Site Scripting (XSS). See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2001-0835
+ /manual/: Directory indexing found.
+ /manual/: Web server manual found.
+ /icons/: Directory indexing found.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /test.php: This might be interesting.
+ /wp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpress/wp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpress/wp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpress/wp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /assets/mobirise/css/meta.php?filesrc=: A PHP backdoor file manager was found.
+ /login.cgi?cli=aa%20aa%27cat%20/etc/hosts: Some D-Link router remote command execution.
+ /shell?cat+/etc/hosts: A backdoor was identified.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8908 requests: 0 error(s) and 30 item(s) reported on remote host
+ End Time:           2025-01-27 09:45:28 (GMT5.5) (30 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
### **Gobuster scan**
```bash
┌──(root㉿KALI)-[/home/hrishi]
└─# gobuster dir -u http://192.168.153.129/ -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -k -t 30 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.153.129/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/mrtg                 (Status: 301) [Size: 292] [--> http://127.0.0.1/mrtg/]
/manual               (Status: 301) [Size: 294] [--> http://127.0.0.1/manual/]
/usage                (Status: 301) [Size: 293] [--> http://127.0.0.1/usage/]
Progress: 141708 / 141709 (100.00%)
===============================================================
Finished
===============================================================
```