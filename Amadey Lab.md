This lab focus on Endpoint Forensics using Volatility 3 (memory forensic) tool, after this lab I understand the usage of volatility tool and investigate using it. The lab is web based, as long as you start the lab, you will see the lab file on Desktop.

Q1. In the memory dump analysis, determining the root of the malicious activity is essential for comprehending the extent of the intrusion. What is the name of the parent process that triggered this malicious behavior?

To answer this question, you can use this command. After you made the analysis about the processes, you will see the anomalies process.

```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.pslist
```

Q2. Once the rogue process is identified, its exact location on the device can reveal more about its nature and source. Where is this process housed on the workstation?

After you found the suspicious process, you will see the PID of the process. Once you found it you can determine its exact location within the file system. `windows.cmdline` plugin in Volatility3, it extracts the command-line arguments that were provided when processes were executed.

```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.cmdline --pid <PID>
```
once this command executed, you will see the Full path of the suscipious process.

Q3. Persistent external communications suggest the malware's attempts to reach out C2C server. Can you identify the Command and Control (C2C) server IP that the process interacts with?

To identify the IP addresss of C2 server, you can use `windows.netscan` plugin, it can analyze the network connections from the memory dump, also it can enumerate active and closed network connections. After that, you will see the C2 server IP address of suscipious process

```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.netscan
```

Q4. Following the malware link with the C2C, the malware is likely fetching additional tools or modules. How many distinct files is it trying to bring onto the compromised workstation?

To answer this one, you needs to use `windows.memmap` plugin, utilized to extract memory regions associated with the process.

```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.memmap --pid <PID> --dump
```

this command will generates a memory dump file. Once the dump is created you can analysis the dump file using `strings`, to extract readable text from memory dump file.

Q5. Identifying the storage points of these additional components is critical for containment and cleanup. What is the full path of the file downloaded and used by the malware in its malicious activity?

Malware often downlaod and executes supporting file like DLL, to expend it capabilties.
For this I used `windows.filescan`, plugin scans memory for file objects and lists their metadata, including file paths.

```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.filescan | grep -E "<FILE _NAME>"
```
After executed this cmd, you will discover the file name that HTTP GET requests made by the suscipious.

Q6. Once retrieved, the malware aims to activate its additional components. Which child process is initiated by the malware to execute these files?

if you answered the question1, you will notice which process of the child process of `lssass.exe` or you can identified using `windows.pstree` plugin.

Q7. Understanding the full range of Amadey's persistence mechanisms can help in an effective mitigation. Apart from the locations already spotlighted, where else might the malware be ensuring its consistent presence?

```
vol -f 'Windows 7 x64-Snapshot4.vmem' windows.filescan | grep "lssass" -E
```
`windows.filescan` plugin, to scan for file artifacs associated with the process. After you executed you will discover the key file locations that associated with the `lssass.exe` process