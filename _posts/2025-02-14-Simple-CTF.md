---
title: "Simple CTF"
date: 2025-02-14T00:00:00+00:00
category: CTF
tags: ["ctf"]
description: "How a Simple CTF took me 2 days to solve"
toc: false
comments: true
---

## Introduction

In this CTF challenge, I was provided with two files: `Wordlist.txt` and `flag.zip`. The `flag.zip` file was password-protected, and my goal was to find the correct password and extract the hidden flag.

## Step 1: Cracking the Zip Password

Since `flag.zip` was password-protected, I decided to use **John the Ripper** to crack it. The steps were:

1. Convert the zip file into a hash format using `zip2john`:

    ```bash
    zip2john flag.zip > flag_hash.txt
    ```

2. Use John the Ripper to brute-force the hash with the given wordlist:

    ```bash
    john --wordlist=Wordlist.txt flag_hash.txt
    ```

3. After some time, John successfully cracked the password. Using the obtained password, I extracted the contents:

    ```bash
    unzip flag.zip
    ```

Inside the extracted folder, I found two files:
- `find_me.jpg`
- `Wordlist.txt` (same as the provided wordlist)

## Step 2: Investigating the Image

At first, I reverse-searched `find_me.jpg` on Google but found no useful clues. I then decided to check its metadata using **ExifTool**:

```bash
exiftool find_me.jpg
```

This revealed an interesting piece of data in the **Author** field:

```
Author                          : U2FsdGVkX1/Nzd+SqTEHDW1boiaehOmCFR0u+S1nQ0ZiYdX5aDGIKa2xADEiS3r/3h+VI4CL8ZLg24l35omqqw==
```

The string looked like **AES-encrypted text**, so I started searching for ways to decrypt it.

## Step 3: Cracking the AES Encryption

After spending about **1.5 days** researching AES decryption, I finally asked ChatGPT to generate a script to brute-force the decryption using the provided wordlist.

Here’s the **JavaScript** script I used to decrypt the text using `CryptoJS`:

```javascript
const fs = require('fs');
const readline = require('readline');
const CryptoJS = require('crypto-js');

function decryptString(encryptedString, password) {
    try {
        const bytes = CryptoJS.AES.decrypt(encryptedString, password);
        const decrypted = bytes.toString(CryptoJS.enc.Utf8);
        if (decrypted) {
            return decrypted;
        }
    } catch (e) {
        return null;
    }
    return null;
}

const fileStream = fs.createReadStream('Wordlist.txt');
const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity
});

const encryptedString = 'U2FsdGVkX1/Nzd+SqTEHDW1boiaehOmCFR0u+S1nQ0ZiYdX5aDGIKa2xADEiS3r/3h+VI4CL8ZLg24l35omqqw==';

rl.on('line', (line) => {
    const password = line.trim();
    const decrypted = decryptString(encryptedString, password);
    if (decrypted) {
        console.log(`Password found: ${password}`);
        console.log(`Decrypted string: ${decrypted}`);
        rl.close();
    }
});

rl.on('close', () => {
    console.log('Finished searching passwords.');
});
```

## Step 4: Running the Script

To execute the script, I ran:

```bash
node bruteforce.js
```

The output revealed:

```
Password found: secret123
Decrypted string: Flag{AES_BruteForce_Works!}
```

## Conclusion

This was a fascinating CTF challenge that involved multiple cryptographic techniques:
- **Brute-forcing zip passwords** using John the Ripper
- **Extracting metadata** from images using ExifTool
- **Decrypting AES-encrypted text** with a brute-force approach

The key takeaway is that **weak passwords make encryption vulnerable**. If you enjoyed this write-up, stay tuned for more CTF challenges and cybersecurity insights!