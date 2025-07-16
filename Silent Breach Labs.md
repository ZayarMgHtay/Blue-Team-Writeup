<img width="1770" height="396" alt="Screenshot from 2025-06-02 09-25-22" src="https://github.com/user-attachments/assets/8f2f804c-5f51-45c3-af4a-0cdffc0d0f09" />

Welcome to my write-up for Silent Breach.

I'm not too familiar with the FTK Imager at first. When I started the lab, I understood the concept of the FTK Imager, CyberChef, and Strings.

In this write-up, I'm going to cover how I analyze in Silent Breach.

The first thing you need to do is read the scenario. You don't need the premium version for this one.

**Questions**

**What is the MD5 hash of the potentially malicious EXE file the user downloaded?**

Once you import the image into the FTK Imager, you will see the MD5 hash for a malicious EXE file.

<img width="876" height="775" alt="Screenshot from 2025-07-16 13-55-08" src="https://github.com/user-attachments/assets/30cfe3df-9f79-4dc0-a43a-8be461b4670d" />

**What is the URL from which the file was downloaded?**

When you find the malicious file, just click, and on the right side, you will see the ZoneTransfer. This is the place where you will see the URL.

<img width="879" height="556" alt="Screenshot from 2025-07-16 13-57-09" src="https://github.com/user-attachments/assets/7294a4be-e034-4069-885c-c70e22cecd4f" />

What application did the user use to download this file?

For this question, I have to analyze which browsers Ethan used, then I discovered that he used Chrome and Edge, and then I focused on extracting and analyzing browser history databases from Chrome and Microsoft Edge.

For Chrome 
```
\AppData\Local\Google\Chrome\User Data\Default\History
```
For MSEdge
```
\AppData\Local\Microsoft\Edge\User Data\Default\History
```

Once you find the `History` files, you can analyst using DB Browser for SQLite. you will see the application the user uses to download malicious file.

By examining Windows Mail artifacts, we found an email address mentioning three IP addresses of servers that are at risk or compromised. What are the IP addresses?

To answer this question, you need to read the resource they give; it has the answer.

You need to find the file "HxStore.hxd", which can be found at the path:
```
\Appdata\Local\Packages\microsoft.windowscommunicationsapps_8wekyb3d8bbwe\LocalState\
```

Once you've found the file, export it.

I used Notepad++ and the HEX Editor plugin. I did a little analysis, and then I found the IP addresses.

<img width="877" height="296" alt="Screenshot from 2025-07-16 13-59-46" src="https://github.com/user-attachments/assets/d0bcf879-0213-43c3-b50b-2fbcf9656828" />

By examining the malicious executable, we found that it uses an obfuscated PowerShell script to decrypt specific files. What predefined password does the script use for encryption?

Now, it's getting a little interesting. For this question, I used the "Strings` tool to analyze the malicious file. I try to find a base64 string in a malicious file. 

Then you will find a malicious PowerShell script designed to execute obfuscated code.
This script uses a **reversed Base64-encoded string**.

<img width="573" height="39" alt="Screenshot from 2025-07-16 14-01-09" src="https://github.com/user-attachments/assets/322c9d5f-2f48-42c2-aefc-3a949577e520" />

<img width="880" height="168" alt="Screenshot from 2025-07-16 14-04-35" src="https://github.com/user-attachments/assets/babc1de7-c307-4978-a517-565944291874" />

I used 2 methods to reverse this. The first one uses `CyberChef`, and the second one uses `echo`. Both are simple.
For the recipe, I used the Reverse and From Base64 functions. Then you will see the answer.

<img width="875" height="504" alt="Screenshot from 2025-07-16 14-01-43" src="https://github.com/user-attachments/assets/36c712e8-4077-4a6b-92ca-8d84dd769b39" />
 
Second Method is using `echo`.
<img width="883" height="228" alt="Screenshot from 2025-07-16 14-02-16" src="https://github.com/user-attachments/assets/7dfb574e-929d-4be6-97e8-b3d6978b4464" /> 

Both methods are simple and effective.

After identifying how the script works, decrypt the files and submit the secret string.

Let's go to the last one. To answer this, you need to know the usage of `openssl`.

If you have finished decoding the script. You need to analyze how the script works.

In the script, you will see, they encrypted the file under this file path: C:\Users\ethan\Desktop. You will see the `.enc` files.

<img width="1517" height="263" alt="Screenshot from 2025-07-16 14-18-37" src="https://github.com/user-attachments/assets/579870da-9989-4b58-a531-82862b3b6bd1" />


There is a simple way to decrypt the previous PowerShell script that encrypts files using **AES encryption** in **CBC mode**, with a key and IV generated using **PBKDF2 (RFC2898)**.

In a PowerShell script it uses a static password and salt. You can reverse and decrypt using the LLM of your choice and ask it to make a decryption script. The hard way is to read up on how system.security.cryptography.AES works, and write your script.

In order to decrypt the script, you already have the  Password, salt, iteration count, and algorithm.

Since the original encryption process did not use OpenSSL's internal salt/header format, OpenSSL would not know how to regenerate the key/IV on its own. Therefore, I had to reproduce the exact key and IV using the same algorithm and parameters.

<img width="876" height="61" alt="Screenshot from 2025-07-16 14-11-12" src="https://github.com/user-attachments/assets/7c4f4de4-b7ce-411f-b6d2-9d530302353d" />

After you run the script, you will get the Key and IV value.

While analyzing a suspicious PowerShell script found in a sample of malware, I discovered that it was designed to encrypt PDF files using **AES-256-CBC encryption**. By reversing the script and replicating the key derivation process, I was able to **decrypt the files successfully** using **OpenSSL**.

Using the derived key and IV, I decrypted the `.enc` file with `openssl`.

To decrypt the encrypted files `IMF-Secret.enc` and `IMF-Mission.enc`, I used the following command:

<img width="883" height="119" alt="Screenshot from 2025-07-16 14-11-50" src="https://github.com/user-attachments/assets/4d433389-5a90-4057-a571-3a03886a34ce" />


To verify the decryption, I used.
```
file IMF-Secret.pdf
file IMF-Mission.pdf
```

Once you open the PDF file, you will find the flag.

This analysis highlights how malware authors used standard cryptographic techniques with hardcoded values. Because they did not randomize the password or salt, it was possible to derive the key and IV and reverse the encryption using OpenSSL.

This exercise reinforced the importance of:
- Understanding how cryptographic functions are implemented in PowerShell
- Being able to translate them to standard tools like OpenSSL
- Recognizing poor cryptographic hygiene in malware
