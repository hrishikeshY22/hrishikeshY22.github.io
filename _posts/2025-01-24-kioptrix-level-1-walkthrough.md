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