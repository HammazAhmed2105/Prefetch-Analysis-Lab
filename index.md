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

```POWERSHELL
C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe  -f C:\Cases\Prefetch\BURPSUITE-PRO-CRACKED.EXE-EF7051A8.pf
```
1. How many times did this program run?
-  ```powershell
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
- Run count: 1
- Last run: 2024-03-12 18:55
2. What is the full path to the program executable?
- \WINDOWS\TEMP\B.EXE
3. User Data this program was accessing?
```powershell
WINDOWS\WEBCACHE\WEBCACHEV01.DAT
USER DATA\DEFAULT\HISTORY
EDGE\USER DATA\DEFAULT\HISTORY
MOZILLA\FIREFOX\PROFILES.INI
```

















   
   





