# Prefetch Analysis of Malware Intrusion

# Note - For detailed step by step analysis, read the index.md file.


```markdown
# Prefetch Analysis of Malware Intrusion 

## Overview
This project investigates a suspected malware intrusion on Bill Lumbergh's workstation at Initech Software by analyzing Prefetch files. The goal is to identify malicious executables, directories accessed, and suspicious activities.

## Scenario
Bill downloaded a cracked version of Burp Suite Pro, leading to potential malware infection. Prefetch files from his system are analyzed to identify the malicious actions.

## Lab Setup
1. **Virtual Machine:** Windows 11 in VirtualBox/VMware.
2. **Tools:** Eric Zimmerman's forensic tools.
   ```powershell
   IEX (New-Object Net.WebClient).DownloadString("https://ec-blog.s3.us-east-1.amazonaws.com/DFIR-Lab/PF_Lab/prep_lab.ps1")
   ```
3. **Directory Structure:** 
   - Prefetch files in `C:\Cases\Prefetch`
   - Tools in `C:\DFIR_Tools\Zimmerman Tools\net6`

## Analysis Steps
1. **Parse Prefetch Files:**
   ```powershell
   PECmd.exe -q -d C:\Cases\Prefetch\ --csv "C:\Cases\Analysis\ --csvf prefetch.csv"
   ```
2. **Timeline Analysis:** Use Timeline Explorer to review execution sequences.
3. **File Investigation:** Search for "burpsuite" and other suspicious files (`7ZG.EXE`, `B.EXE`, `C.EXE`).
4. **Detailed File Analysis:** Extract run counts and paths using `PECmd.exe`.

## Findings
- **BurpSuite-Pro-Cracked.exe:** Executed once, indicating malware launch.
- **B.EXE, C.EXE:** Accessed browser and credential files.
- **Powershell.exe:** High execution count; accessed company data.
- **Rclone.exe:** Potentially used for data exfiltration.

## Conclusion
Bill's system was compromised through the cracked Burp Suite download, leading to malware executing various tools for data gathering and exfiltration.

## Tools Used
- Eric Zimmerman's `PECmd.exe`
- Timeline Explorer
```


