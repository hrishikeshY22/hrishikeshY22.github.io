---
title: "CloudSEK 2025 Hiring CTF Round 2 Writeup"
date: 2025-12-20T00:00:00+00:00
category: CTF
tags: ["ctf", "cloudsek", "jwt", "ssti"]
description: "Walkthrough of CloudSEK Hiring CTF Round 2 – JWT abuse, checksum bypass, and SSTI to read flag.txt."
toc: true
comments: true
---
---
## Introduction

![](/assets/img/posts/CloudSek%20Round2/start.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

In Round 2 of the CloudSEK Hiring CTF, the goal was to pivot from an operator role to an admin role in a flight control‑themed web application, bypass client‑side integrity checks, and ultimately exploit an SSTI vulnerability to read `flag.txt` from `/root`.


---

## Step 1 – Finding Hardcoded Credentials

1. Configure the browser to use **Burp Suite** as an intercepting proxy and browse the challenge application.
2. While browsing, static resources are loaded from `/static/js/`. Among them, `secrets.js` contains an `operatorLedger` array with hardcoded credentials.
3. Extract the operator account from the script:

    ```
    flightoperator : GlowCloud!93
    ```

    ![](/assets/img/posts/CloudSek%20Round2/step1.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
4. Use these credentials on the login endpoint `/api/login` to obtain a valid JWT token.

---

## Step 2 – Cracking and Forging the JWT

1. Capture the login request and response in Burp to obtain the JWT returned by `/api/login`.
2. Use `jwt_tool` to analyze and crack the token’s HMAC secret key.The tool reveals the signing secret is:

    ![](/assets/img/posts/CloudSek%20Round2/jwt%20crack.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

3. Decode the token at `jwt.io` or using `jwt_tool` and observe that the payload contains a `role` field set to `operator`.
4. Modify the payload:

    ```json
    {
      "sub": "flightoperator",
      "role": "admin",
      ...
    }
    ```

5. Re‑sign the JWT with algorithm `HS256` and secret `butterfly`, generating a forged admin token.
6. Replace the original JWT in the browser (cookies or `Authorization` header) with the forged admin JWT and refresh `/console`. The application now exposes the **Quantum Admin Beacon** panel.

    ![](/assets/img/posts/CloudSek%20Round2/admin.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

---

## Step 3 – Bypassing Client-Side Checksum Validation

1. In the admin console, actions send requests to `/api/admin/hyperpulse` with a JSON payload and a checksum. 
2. Inspect `/static/js/console.js` and locate the function:

    ```js
    function computeChecksum(payload, token) {
        // custom checksum logic here
    }
    ```

   This function computes a checksum over the request payload and the JWT token, which the server verifies. 

3. Reverse‑engineer the checksum logic from `console.js` and implement an equivalent Python function to generate valid checksums for custom payloads. 

    ```python
    def compute_checksum(payload, token):
    buffer = f"{payload}::{token}"
    acc = 0x9e3779b1
    for i in range(len(buffer)):
        code = ord(buffer[i])
        shift = i % 5
        acc ^= (code << shift) + (code << 12)
        acc = (acc + ((acc << 7) & 0xFFFFFFFF)) ^ (acc >> 3)
        acc &= 0xFFFFFFFF
        acc ^= (acc << 11) & 0xFFFFFFFF
        acc &= 0xFFFFFFFF
    return format(acc, "08x")

    token = ""
    print(compute_checksum("", token))
    ```

4. With this, crafted admin payloads can be sent directly via Burp Repeater or a custom script while still passing checksum validation. 

---

## Step 4 – Exploiting SSTI in the Admin Endpoint

1. Start testing payload fields in `/api/admin/hyperpulse` for injection. Commands and template‑like strings cause different responses, indicating server‑side rendering. 
2. Inject {7*7} into a controllable field 'message' in the JSON body and recompute the checksum before sending.
3. The API response returns `49` in the `result` field, confirming a **Server-Side Template Injection (SSTI)** vulnerability. 
4. 
![](/assets/img/posts/CloudSek%20Round2/ssti.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
---

## Step 5 – Listing `/root` and Reading `flag.txt`

1. Use SSTI to interact with the underlying runtime and list `/root`. For a Jinja2‑like environment, a typical pattern is to use object traversal to reach `os` and call `listdir`. 
   
   ![](/assets/img/posts/CloudSek%20Round2/list.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

   (Exact gadget chain depends on the template engine; the idea is to use SSTI to execute `os.listdir`.) 

2. Recompute the checksum with your Python `compute_checksum` function and send the request. The response shows that `/root` contains `flag.txt`. 
3. Craft another SSTI payload to read the file content:
    
    ![](/assets/img/posts/CloudSek%20Round2/flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

4. Again, recompute the checksum and send the request. The `result` field in the response now contains the flag from `flag.txt`. 

---

## Conclusion

This challenge chained several real‑world vulnerabilities:

- Exposed **client‑side secrets** in JavaScript.
- Weakly protected **JWT** tokens that could be brute‑forced and forged.
- A fragile **client‑side checksum** used as a “security” control.
- A critical **SSTI** primitive that ultimately allowed arbitrary file reads from `/root`.