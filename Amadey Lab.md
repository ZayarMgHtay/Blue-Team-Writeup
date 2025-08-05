### This lab focuses on Endpoint Forensics using Volatility 3 (memory forensic) tool. After completing this lab, you will gain an understanding of the volatility tool and have begun to investigate its usage. The lab is web-based; as long as you start the lab, you will see the lab file under the Desktop directory.

### Q1. In the memory dump analysis, determining the root of the malicious activity is essential for comprehending the extent of the intrusion. What is the name of the parent process that triggered this malicious behavior?

To answer this question, you can use this command. After you have analyzed the processes, you will see the anomaly process.
```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.pslist
```
<img width="1884" height="262" alt="2025-08-05_22-03" src="https://github.com/user-attachments/assets/e073f20e-a63a-4c3b-b747-c7b5aef68731" />

### Q2. Once the rogue process is identified, its exact location on the device can reveal more about its nature and source. Where is this process housed on the workstation?

After you have found the suspicious process, you will see the PID of the process. Once you have found it, you can determine its exact location within the file system. `windows.cmdline` plugin in Volatility3 extracts the command-line arguments that were provided when processes were executed.
```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.cmdline --pid <PID>
```
Once this command is executed, you will see the Full path of the suspicious process.

<img width="1422" height="152" alt="2025-08-05_22-05" src="https://github.com/user-attachments/assets/90eed915-57ff-42f3-bb3e-acf90887a64b" />

### Q3. Persistent external communications suggest the malware's attempts to reach out C2C server. Can you identify the Command and Control (C2C) server IP that the process interacts with?

To identify the IP addresses of C2 server, you can use `windows.netscan` plugin, which can analyze the network connections from the memory dump, and also it can enumerate active and closed network connections. After that, you will see the C2 server IP address of the suspicious process
```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.netscan
```
<img width="1499" height="128" alt="2025-08-05_22-06" src="https://github.com/user-attachments/assets/ba561342-3871-460f-a851-671bd2fc76c7" />

### Q4. Following the malware link with the C2C, the malware is likely fetching additional tools or modules. How many distinct files is it trying to bring onto the compromised workstation?

To answer this one, you need to use `windows.memmap` plugin, utilized to extract memory regions associated with the process.
```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.memmap --pid <PID> --dump
```
This command will generate a memory dump file. Once the dump is created, you can analyze the dump file using `strings` to extract readable text from the memory dump file.

<img width="686" height="77" alt="2025-08-05_22-08" src="https://github.com/user-attachments/assets/56894023-4d82-43b7-bb78-a83c01147800" />

<img width="1048" height="73" alt="2025-08-05_22-09" src="https://github.com/user-attachments/assets/0e865292-03ee-4172-82c3-be2d3d3a1faa" />

### Q5. Identifying the storage points of these additional components is critical for containment and cleanup. What is the full path of the file downloaded and used by the malware in its malicious activity?

Malware often downloads and executes supporting files, such as DLLs and scripts, to expand its capabilities.
For this, I used `windows.filescan`, a plugin that scans memory for file objects and lists their metadata, including file paths.
```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.filescan | grep -E "<FILE _NAME>"
```
<img width="1604" height="53" alt="2025-08-05_22-11" src="https://github.com/user-attachments/assets/83dcf0c1-4523-441c-ae3f-4b50e3057fdc" />

After executing this command, you will discover the file name that HTTP GET requests were made by the suspicious.

### Q6. Once retrieved, the malware aims to activate its additional components. Which child process is initiated by the malware to execute these files?

If you answered question 1, you will notice which process of the child process of `lssass.exe`, or you can identify it using `windows.pstree` plugin.

<img width="1439" height="263" alt="2025-08-05_22-13" src="https://github.com/user-attachments/assets/cd4207cd-cbe0-44fe-a9e9-792614951e23" />

### Q7. Understanding the full range of Amadey's persistence mechanisms can help in effective mitigation. Apart from the locations already spotlighted, where else might the malware be ensuring its consistent presence?
```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.filescan | grep "lssass" -E
```
<img width="1518" height="100" alt="2025-08-05_22-12" src="https://github.com/user-attachments/assets/fe661660-026b-497b-82ae-45aafd97de7a" />

`windows.filescan` plugin, to scan for file artifacts associated with the process. After you execute, you will discover the key file locations associated with the `lssass.exe` process.
