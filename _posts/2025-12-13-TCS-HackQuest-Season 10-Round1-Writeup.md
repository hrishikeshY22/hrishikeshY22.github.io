---
title: "TCS HackQuest Season 10 – Round 1 Writeup"
date: 2025-12-16T00:00:00+00:00
category: CTF
tags: ["ctf", "hackquest"]
description: "Writeup for TCS HackQuest Season 10 Round 1 – 8 challenges solved out of 14."
toc: true
comments: true
---


## AREA 64

**Flag:** `HQX{70c5d525ca6fa443894989a49bf4ccf8}`  

### Approach

1. Download the task ZIP file and extract it.
2. Inside, open the text file `4DFB3A68bb.txt`.
3. The file contains a Base64‑encoded string:  
   `WW91J3JlIGluc2lkZSBBcmVhIDY0LiBIZXJlJ3MgeW91ciBrZXkgOiBIUVh7NzBjNWQ1MjVjYTZmYTQ0Mzg5NDk4OWE0OWJmNGNjZjh9`
4. Decode the string using Base64 to reveal the flag.
![](/assets/img/posts/TCS%20Hackquest%20Round%201/Area%2064%20Flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }
![](/assets/img/posts/TCS%20Hackquest%20Round%201/Area%2064%20Submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

## Small-E

**Flag:** `HQX{36b8682966e652532a191b3f687a77ff}`  

### Approach

1. Download the challenge archive and unzip it.
2. Inspect the Python file; it shows RSA encryption with a very small public exponent `e`.
3. Rewrite the script to directly compute the integer cube root of the ciphertext and convert it back to bytes:

    ```python
    n = <given_n_value>
    e = <given_e_value>  # small, e.g. 3
    ct = <given_c_value>

    def icbrt_newton(n):
        if n < 1:
            return n
        x = 1 << ((n.bit_length() + 2) // 3)
        while True:
            y = (2 * x + n // (x * x)) // 3
            if y >= x:
                return x
            x = y

    def long_to_bytes(n):
        s = b""
        while n > 0:
            s = bytes([n & 0xff]) + s
            n >>= 8
        return s

    m = icbrt_newton(ct)
    flag_bytes = long_to_bytes(m)
    flag = flag_bytes.decode("ascii")
    print("Flag:", flag)
    ```

4. Run the script to recover the plaintext and flag.
   
    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/small%20E%20flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

## Hidden Layers

**Flag:** `HQX{24c0ce09e09ab4caa1b8e8439d1d48f0}`  

### Approach

1. Download and extract the ZIP file.
2. The archive contains an image; upload it to an online LSB/steganography tool such as `https://stylesuxx.github.io/steganography/`.
3. Decode the embedded message inside the image to obtain the flag.

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/hidden%20layers%20submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }


## Unfair Flip

**Flag:** `HQX{5c4e92253474ebfd10b1b00950e86a41}`  

### Approach

1. Open the web challenge in a browser and open the developer console.
2. The game logic uses a `coins` array and a hidden flag function.
3. In the console, force all coins to heads and invoke the hidden flag function:

    ```javascript
    window.coins = ["H", "H", "H"];
    _hiddenFlag();
    ```

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/unfair%20flip%20flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/unfair%20flip%20submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

## Refresh Ritual

**Flag:** `HQX{334fe20f2594f380e2fbb20c5412d3dc}`  

### Approach

1. Open the web‑based challenge.
2. Inspect the HTML/JS of the page using developer tools.
3. The password input field has a hint value in the source that directly corresponds to the correct password.
4. Enter that hint value as the password; the page reveals the flag.

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/refresh%20ritual%20flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/Refresh%20ritual%20submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

## Seeds of Time

**Flag:** `HQX{c126bb454bc9e067bdcf0cf0729fbd08}`  

### Approach

1. Download the ZIP file and extract the contents.
2. The given code shows an encryption that XORs the plaintext with a PRNG keystream seeded by `time.time()`. The ciphertext is provided as hex.
3. Reverse the process by brute‑forcing possible seeds around the current time:

    ```python
    import random
    import time

    cipher_hex = "f60d1ef6307bc56ed4f3f8fe41eaccc4d5bd24aadc8f9f834cb2083b601303b923ebba81ca"
    cipher = bytes.fromhex(cipher_hex)

    now = int(time.time())

    for seed in range(now - 100000, now + 1):
        random.seed(seed)
        keystream = bytearray(int(random.random() * 256) for _ in range(len(cipher)))
        plaintext = bytes(c ^ k for c, k in zip(cipher, keystream))

        try:
            text = plaintext.decode("utf-8")
            if all(32 <= ord(c) <= 126 for c in text):
                print(f"[+] Seed: {seed}")
                print(f"[+] Flag: {text}")
                break
        except:
            continue
    ```

4. Execute the script and read the flag from the decoded plaintext.

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/seeds%20of%20time%20submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }


## Synthetic Stacks

**Flag:** `HQX{df30cb178e42f66815417e3e6551c8de}`  

### Approach

1. Download and extract the ZIP file; it contains a `.png` file.
2. Inspect the PNG header and hex data and notice that it actually contains a 7‑Zip archive signature.
3. Rename the file from `.png` to `.7z`.
4. Extract the `.7z` archive with 7‑Zip; it prompts for a password.
5. Use `7z2john` to generate a hash and crack it with `john`:

    ```
    7z2john challenge.7z > hash.txt
    john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
    ```

   The recovered password is `October`.
6. Use the password to extract `hq.txt`, which contains Base64‑encoded data. Decode it and save the result as a PNG file.
7. The new PNG shows a QR code. Scan the QR code to reveal the flag.

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/Synthetic%20stacks%20flag.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/Synthetic%20stacks%20submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }


## Address Abyss

**Flag:** `HQX{e1f63411d00a04a5d80803bb0860d9f4}`  

### Approach

1. Download and extract the ZIP file; it contains a large text file with many IP‑like entries.
2. The challenge description gives conditions that map parts of IPv4 and IPv6 addresses to positions in the flag.
3. Write a Python script using regular expressions to parse each line, build a mapping, and reconstruct the flag:

    ```python
    import re

    pairs = {}

    ipv4 = re.compile(r"^92\.7\.(\d+)\.(\d)$")
    ipv6 = re.compile(r"^2510:a1:([0-9a-fA-F]+)::([0-9a-fA-F])$")

    with open("ip_logs_4DFB3A68bb.txt") as f:
        for line in f:
            line = line.strip()

            m4 = ipv4.match(line)
            if m4:
                idx = int(m4.group(1))
                val = m4.group(2)
                pairs[idx] = val
                continue

            m6 = ipv6.match(line)
            if m6:
                idx = int(m6.group(1), 16)
                val = m6.group(2)
                pairs[idx] = val

    flag = ''.join(pairs[i] for i in sorted(pairs))
    print("HQX{" + flag + "}")

    ```

4. Run the script to print the full flag.

    ![](/assets/img/posts/TCS%20Hackquest%20Round%201/address%20abyss%20submit.png){: style="display:block; margin-left:auto; margin-right:auto; width:80%;" }


## Conclusion

HackQuest 10 Round 1 covered a wide range of concepts: base64 decoding, low‑exponent RSA, PRNG‑based stream ciphers, steganography, regex‑driven parsing, archive and file‑format tricks, and basic web exploitation. Working through these challenges reinforced how important it is to read the problem statement carefully, trust your tooling, and be ready to pivot between crypto, web, and forensics techniques.

Overall, solving 8 challenges for 1400 points was a solid learning experience and a fun test of problem‑solving under time pressure. If you found this writeup useful or have alternate approaches to any of the tasks, feel free to reach out or share your methods—there’s always more to learn from different perspectives.
