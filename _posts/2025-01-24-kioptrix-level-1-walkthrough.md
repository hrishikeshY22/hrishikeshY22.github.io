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