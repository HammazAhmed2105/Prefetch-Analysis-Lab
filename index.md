# Scenario
We are investigating an intrusion involving a workstation owned by Bill Lumbergh of the Initech Software company. Bill is currently an IT technician hoping to break into the exciting cybersecurity career field.

Recently, Bill was looking for free resources for testing his skills in web app penetration testing and used Reddit to try to find a cracked version of a popular software called Burpsuite Pro. Unfortunately, an unsavory Redditor may have sent Bill some malware.

We have acquired key forensic artifacts from Bill’s system to better understand what happened once he ran the malware. This lab focuses exclusively on the Prefetch files obtained from Bill’s system. 

# Lab Setup

1. First things first, we need to set up a virtual machine to download the forensic evidence. I am using Oracle VirtualBox with a Windows 11 VM. You can also use VMware from [this link](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion) and obtain a Windows 11 VM from [this link](https://www.microsoft.com/en-us/software-download/windows11).

2. We will download all of Eric Zimmerman's tools, which will help us analyze the prefetch files. The link will also download all the forensic evidence, DotNet 6, and the necessary EZ tools. Use the following command to download everything:

   ```powershell
   IEX (New-Object Net.WebClient).DownloadString("https://ec-blog.s3.us-east-1.amazonaws.com/DFIR-Lab/PF_Lab/prep_lab.ps1")
   ```
3. Once the script is finished running, we will see the below ouput. In the 2nd screenshot you can also see , we have got the prefetch files too.

<img src="https://i.imgur.com/58LevzJ.png" height="65%" width="65%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/pWspxoZ.png" height="65%" width="65%" alt="Disk Sanitization Steps"/>
4. The prefetch Files can be found in **C:\Cases\Prefetch**, while the EZ tools are in **C:\DFIR_Tools\Zimmerman Tools\net6**.

## Note
1. Our goal is to look at the malicious executables that ran, which directories they accesses, how many times they were ran, and a bit more.

Here’s the properly structured content formatted for GitHub:

```markdown
# Analysis and Steps

1. **Parse Prefetch Files**

   Our first step is to use `PECmd.exe` to parse all the prefetch files we have into a CSV, so we can transfer those logs to an Eric Zimmerman tool called Timeline Explorer.

   ```powershell
   C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe -q -d C:\Cases\Prefetch\ --csv "C:\Cases\Analysis\ --csvf prefetch.csv"
   ```

   <img src="https://i.imgur.com/ypuSCL9.png" height="65%" width="65%" alt="Disk Sanitization Steps"/>

   This command runs the `PECmd.exe` tool from Eric Zimmerman's suite with the following options:

   - `-q`: Enables quiet mode (minimal output).
   - `-d C:\Cases\Prefetch\`: Specifies the directory containing Prefetch files to analyze.
   - `--csv "C:\Cases\Analysis\ --csvf prefetch.csv"`: Exports the analysis results to a CSV file named `prefetch.csv` in the `C:\Cases\Analysis\` directory.

   The tool processes the Prefetch files and saves the results in a CSV format for further analysis.

2. **Open Timeline Explorer**

   Open Timeline Explorer from the Desktop, and open the two newly created CSV files.

   <img src="https://i.imgur.com/h99tq4s.png" height="65%" width="65%" alt="Timeline Explorer Screenshot"/>

   <img src="https://i.imgur.com/6eywqw1.png" height="65%" width="65%" alt="Timeline Explorer Screenshot"/>

3. **Sort the Timeline**

   Click on the `RunTime` column to sort the timeline in either ascending or descending order. This helps in creating a proper timeline of events.

4. **Search for Specific Terms**

   According to our scenario, Bill downloaded Burpsuite. Search for the term in Timeline Explorer.

   - We find a single hit on the term `burpsuite`, located at:
     ```powershell
     USERS\BILL.LUMBERGH\DOWNLOADS\BURPSUITE-PRO-CRACKED.EXE
     ```

   <img src="https://i.imgur.com/UbPl3l9.png" height="65%" width="65%" alt="Burpsuite Download Location"/>

   - It's evident Bill downloaded a cracked version of the Burpsuite application on:
     ```powershell
     2024-03-12 at 18:36🕚
     ```

5. **Analyze Nearby Logs**

   Clear the search term for Burpsuite, and now analyze the nearby logs. Just before Burpsuite was installed, a file called `7ZG.EXE` was executed, which is potential malware.

   <img src="https://i.imgur.com/uJCPhzZ.png" height="65%" width="65%" alt="Malware Execution"/>


6. **Identify Suspicious Files**

   Scroll through the logs to identify any suspicious files. We come across a few as shown below:

   <img src="https://i.imgur.com/GysrugE.png" height="65%" width="65%" alt="Suspicious Files"/>
   <img src="https://i.imgur.com/akoJ51H.png" height="65%" width="65%" alt="Suspicious Files"/>
   <img src="https://i.imgur.com/xG9WuO4.png" height="65%" width="65%" alt="Suspicious Files"/>

   - **schtasks.exe**: Might be used for persistence.
   - **B.EXE, C.EXE, P.EXE**: Unfamiliar names and residing in TEMP.
   - **WHOAMI.EXE**: Displays user, group, and privileges information. Commonly used by attackers.


7. Analyzing each prefetch files
   In order to confirm how burpsuite is related to 7ZG.EXE, we will try to look at the 7GZ.EXE prefetch file, and search for the word "burpsuite", to see how it is referenced. In the below command, -k helps us search for the word burpsuite.

   ```powershell
      C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe -k burpsuite -f C:\Cases\Prefetch\7ZG.EXE-D9AA3A0B.pf
   ```

    <img src="https://i.imgur.com/C2hJoIE.png" height="65%" width="65%" alt="Suspicious Files"/>

   <img src="https://i.imgur.com/JXagiSf.png" height="65%" width="65%" alt="Suspicious Files"/>

We do see a file called **BURPSUITE-PRO-CRACKED.7Z**, we can conclude that it's the same file that extracted **burpsuite-pro-cracked.exe**.

8. Analyzing other suspcious files.

Now, we will analyze the remaining files that we found using PECmd.exe and answer the below questions.
- How many times did this program run?
- What is the full path to the program executable?
- Anything interesting or noteworthy in the Files or Directories referenced?


## File to Analyze - BURPSUITE-PRO-CRACKED.7Z

```powershell
C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe  -f C:\Cases\Prefetch\BURPSUITE-PRO-CRACKED.EXE-EF7051A8.pf
```

1. How many times did this program run?
```powershell
   Run count: 1
Last run: 2024-03-12 18:36:11
```
2. What is the full path to the program executable?
```powershell
\USERS\BILL.LUMBERGH\DOWNLOADS\BURPSUITE-PRO-CRACKED.EXE
```
3. Anything interesting or noteworthy in the Files or Directories referenced?
- Theres no actual indicator of compromise, but we have **WININET.DLL**, this is interesting since it accesses internet sources, which can be an indicator that its trying to communicate with a Command and Control Server.

  ## File to Analyze - B.EXE.pf
```powershell
C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe  -f C:\Cases\Prefetch\B.EXE.pf
```
1. How many times did this program run?
```powershell
- Run count: 1
- Last run: 2024-03-12 18:55
```
2. What is the full path to the program executable?
```powershell
- \WINDOWS\TEMP\B.EXE
```
3. User Data this program was accessing?
```
WINDOWS\WEBCACHE\WEBCACHEV01.DAT
USER DATA\DEFAULT\HISTORY
EDGE\USER DATA\DEFAULT\HISTORY
MOZILLA\FIREFOX\PROFILES.INI
```

 ## File to Analyze - C.EXE.pf
```powershell
C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe  -f C:\Cases\Prefetch\C.EXE.pf
```
1. How many times did this program run?
```powershell
- RunCount: 9
- Last run: 2024-03-12 19:02:37
```
2. What is the full path to the program executable?
```powershell
- \WINDOWS\TEMP\C.EXE
```
3. Direct Key word hits?
```powershell
- \WINDOWS\TEMP\2.TXT
- \WINDOWS\TEMP\WCEAUX.DLL
```
### Note - WCEAUX.DLL is definitely suspicious since its usually involved in pass the hash or pass the ticekt attacks. Check the below link for reference.
https://jpcertcc.github.io/ToolAnalysisResultSheet/details/RemoteLogin-WCE.htm

## File to Analyze - POWERSHELL.EXE.pf

1. How many times did this program run?
```powershell
Run count: 23
Last run: 2024-04-13 21:31:28
Other run times: 2024-04-13 21:21:23, 2024-04-13 21:21:22, 2024-04-13 20:50:40, 2024-04-13 20:50:40, 2024-03-12 19:26:55, 2024-03-12 19:16:52, 2024-03-12 19:14:15
```

```
2. Interesting Directories Accessed?
- We definitely see a lot of copany files being copied in the \BACKUP\LOGS\. This is suspicious since all these are business files and might be confidential.
```powershell
\WINDOWS\SYSTEM32\NTMARTA.DLL
\USERS\BILL.LUMBERGH\DESKTOP\IT DOCS\ACCOUNTS-EXPORT-2023-07-24.XLS
\WINDOWS\BACKUP\LOGS\ACCOUNTS-EXPORT-2023-07-24.XLS
\USERS\BILL.LUMBERGH\DESKTOP\IT DOCS\CYBER-INSURANCE-POLICY-2023.PDF
\WINDOWS\BACKUP\LOGS\CYBER-INSURANCE-POLICY-2023.PDF
USERS\BILL.LUMBERGH\DESKTOP\IT DOCS\DC-BACKUPS.ZIP
\WINDOWS\BACKUP\LOGS\DC-BACKUPS.ZIP
\USERS\BILL.LUMBERGH\DESKTOP\IT DOCS\IT-SYSTEMS-DIAGRAM.PDF
\BACKUP\LOGS\IT-SYSTEMS-DIAGRAM.PDF
\USERS\BILL.LUMBERGH\DESKTOP\IT DOCS\OFFSITE BACKUP ARCHITECTURE.PDF
WINDOWS\BACKUP\LOGS\OFFSITE BACKUP ARCHITECTURE.PDF
```

## File to Analyze - Rclone.EXE.pf

- This is one of the most suspicious prefetch files since it can be used to migrate or mirror data.
- As shown below it also accesses, the previously mentioned files from B.EXe, C.EXE, and some business realted pdf files.

<img src="https://i.imgur.com/Ghi96bP.png" height="65%" width="65%" alt="Suspicious Files"/>

<img src="https://i.imgur.com/10iDyE2.png" height="65%" width="65%" alt="Suspicious Files"/>

1. How many times did this program run?
```powershell
- Run Count: 1
- Last Run: 2024-02-12 19:19:48
```
2. What is the full path to the program executable?
```powershell
\WINDOWS\BACKUP\RCLONE.EXE
```
3. Keyword hits?
```powershell
WINDOWS\BACKUP\RCLONE.CONF (Keyword: True)
\WINDOWS\BACKUP\LOGS\1.TXT (Keyword: True)
\WINDOWS\BACKUP\LOGS\IT-SYSTEMS-DIAGRAM.PDF (Keyword: True)
\WINDOWS\BACKUP\LOGS\2.TXT (Keyword: True)
\WINDOWS\BACKUP\LOGS\OFFSITE BACKUP ARCHITECTURE.PDF (Keyword: True)
\WINDOWS\BACKUP\LOGS\ACCOUNTS-EXPORT-2023-07-24.XLS (Keyword: True)
\WINDOWS\BACKUP\LOGS\CYBER-INSURANCE-POLICY-2023.PDF (Keyword: True)
\WINDOWS\BACKUP\LOGS\DC-BACKUPS.ZIP (Keyword: True)
\WINDOWS\BACKUP\LOGS\LSASS.DMP (Keyword: True
```

## File to Analyze - Sd.EXE.pf

- I didn't checkout the prefetch analysis for sd.exe, but witha  simple google search i found a link from anyrun. Theres a possibility that sd.exe was a wiper or secure delete utility.
- https://any.run/report/a0eef815bef0bf90fb507aa767558187573dca2c1ad692e2bf240a21300a7539/027591fc-19eb-4fac-a1df-f77dc7df0211

## Creating a TimeLine 
We will search for each suspicious file on Timeline Explorer, and tag them. This will help us in making a timeline.

<img src="https://i.imgur.com/mcEgt6G.png" height="65%" width="65%" alt="Suspicious Files"/>

# Timeline of Suspicious Events

## 1. Burpsuite Download
- **Date**: 2024-03-12 at 18:36
- **Details**: Bill downloaded a cracked version of Burpsuite Pro, identified as `BURPSUITE-PRO-CRACKED.EXE`.
- **Location**: `USERS\BILL.LUMBERGH\DOWNLOADS\BURPSUITE-PRO-CRACKED.EXE`

## 2. Malware Execution
- **Date**: 2024-03-12 at 18:55
- **Details**: File `7ZG.EXE` was executed. This is identified as potential malware and was run shortly after the Burpsuite download.

## 3. Execution of Suspicious Files
- **Date**: 2024-03-12 at 19:02
- **Details**: File `C.EXE` was executed multiple times, with a notable run at this time.
- **Location**: `\WINDOWS\TEMP\C.EXE`

## 4. Additional Suspicious Activity
- **Date**: 2024-03-12 at 19:26
- **Details**: `POWERSHELL.EXE` was run, showing high activity and accessing various business files, suggesting possible exfiltration or data manipulation.

## 5. Rclone Execution
- **Date**: 2024-02-12 at 19:19
- **Details**: `RCLONE.EXE` was executed. This tool can be used for migrating or mirroring data and accessed multiple sensitive files, including business-related documents.

## 6. Analysis of Files in TEMP
- **Date**: 2024-03-12 at various times
- **Details**: Files like `B.EXE`, `C.EXE`, and `RCLONE.EXE` accessed and potentially manipulated files in the TEMP directory and other sensitive areas.


# Summary
A cracked version of Burpsuite Pro (`BURPSUITE-PRO-CRACKED.EXE`) was downloaded by Bill on 2024-03-12. Shortly after, potential malware (`7ZG.EXE`) was executed, followed by the repeated execution of a suspicious file (`C.EXE`) from the `\WINDOWS\TEMP\` directory. Later, high activity from `POWERSHELL.EXE` indicated potential data exfiltration or manipulation. Prior to these events, on 2024-02-12, `RCLONE.EXE` was executed, accessing sensitive business documents, suggesting data transfer activities. Files like `B.EXE`, `C.EXE`, and `RCLONE.EXE` also manipulated files in the TEMP directory, indicating widespread suspicious activity.
   





