![[Screenshot from 2025-06-02 09-25-22.png]]

Welcome to my write-up for Slient Breach.

I'm not too familar with the FTK Imager at first. When I started the lab, I do understand the concept of the FTK Imager, CyberChef, and Strings.

In this write-up, I'm going to cover how I analyst in Slient Breach.

That first you need to do is read the scenario. You don't need premium version for this one.

**Questions**

**What is the MD5 hash of the potentially malicious EXE file the user downloaded?**

Once you import the image to the FTK Imager you will see the MD5 hash for malicious EXE file.

![[2025-06-02_09-48.png]]

**What is the URL from which the file was downloaded?**

when you found the malicious file, just click and on the right side you will see the ZoneTransfer. This is the place you will see the URL.

![[2025-06-02_09-54.png]]

What application did the user use to download this file?

For this question, I have to analyst which brower ethan used, then I discovered that he used Chrome and Edge, then I focus on extracting and analyzing browser history databases from Chrome, and Microsoft Edge.

For Chrome I
```
\AppData\Local\Google\Chrome\User Data\Default\History
```

```
\AppData\Local\Microsoft\Edge\User Data\Default\History
```

Once you find the `History` files you can analyst using DB Browser for SQLite. you will see the application the user uses to download malicious file.

By examining Windows Mail artifacts, we found an email address mentioning three IP addresses of servers that are at risk or compromised. What are the IP addresses?

To answer this question you needs to read the resource they give, it have the answer.

You needs to find the file "HxStore.hxd", can be found at path:
```
\Appdata\Local\Packages\microsoft.windowscommunicationsapps_8wekyb3d8bbwe\LocalState\
```

Once you found the file, export it.

I used notepad++, and used the HEX Editor plugin. I did little analyst, then I found the IP addresses.

![[2025-06-02_10-07.png]]

By examining the malicious executable, we found that it uses an obfuscated PowerShell script to decrypt specific files. What predefined password does the script use for encryption?

Now, It getting a little interest. For this questions I used "`Strings` tool for analyst the malicious file. I try to find base64 string in malicious file. 

Then you will find malicious Powershell script designed t execute obfuscated code.
This script use **reversed Base64-encoded string**.

![[Screenshot from 2025-06-02 10-19-05.png]]

![[2025-06-02_10-15.png]]
I used 2 methods to reverse this. This first one is using `CyberChef` and the second one is the using `echo`. Both is simple one if you understands.

![[2025-06-02_10-22.png]]
For recipe, I used Reverse and From Base64. Then you will see the answer.

Second Method is using `echo`.
![[2025-06-02_10-26.png]] Both methods are simple and effective.

After identifying how the script works, decrypt the files and submit the secret string.

Let go to the last one. In order to answer this one, you need to know usage about `openssl`.

If you done finish to decoded script. you needs to analyst that how script works.

In script you will see, they encrypted the file under this file path C:\Users\ethan\Desktop. you will see the `.enc` files.

![[2025-06-02_10-34.png]]

there is easy way to decrypt previous powershell script that encrypts files using **AES encryption** in **CBC mode**, with a key and IV derived using **PBKDF2 (RFC2898)**.

In powershell script, It uses a static password and salt. You can reverse and decrypt using LLM of your choice and ask it to make a decryption script. The hard way is read up how system.security.cryptography.AES works and write your own script.

In order to decrypt the script you already have Password, salt, iteration count and algorithm.

Since the original encryption process did not use OpenSSL's interna salt/header format, OpenSSL would not know how to regenerate the key/IV on its own. Therefore, I had to reproduce the exact key and IV using the same algorithm and parameters.

![[2025-06-07_19-32.png]]
After you ran the script you will get the Key and IV value.

While analyzing a suspicious PowerShell script found in a sample of malware, I discovered that it was designed to encrypt PDF files using **AES-256-CBC encryption**. By reversing the script and replicating the key derivation process, I was able to **decrypt the files successfully** using **OpenSSL**.

Using the derived key and IV, I decrypted the `.enc` file with `openssl`.

To decrypt the encrypted file `IMF-Secret.enc` and `IMF-Mission.enc`, I used the following command:

![[2025-06-07_19-35.png]]

![[2025-06-07_19-37.png]]

To verify the decryption, I used.

```
file IMF-Secret.pdf
file IMF-Mission.pdf
```

Once you open the pdf file, you will find the flag.

This analysis highlights how malware authors used standard cryptographic techniques with hardcoded values. Because they did not randomize the password or salt, it was possible to derive the key and IV and reverse the encryption using OpenSSL.

This exerciese reinforced the important of:
- Understanding how cryptographic functions are implemented in Powershell
- Being able to translate them to standard tools like OpenSSL
- Recognizing poor cryptograhic hygiene in malware