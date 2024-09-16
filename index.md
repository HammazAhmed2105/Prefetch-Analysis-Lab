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

<img src="https://i.imgur.com/58LevzJ.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/pWspxoZ.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>
4. The prefetch Files can be found in **C:\Cases\Prefetch**, while the EZ tools are in **C:\DFIR_Tools\Zimmerman Tools\net6**.

## Note
1. Our goal is to look at the malicious executables that ran, which directories they accesses, how many times they were ran, and a bit more.

# Analysis and Steps

1. Our first step is to use PECmd.exe to parse all the prefetch files we have into a CSV, so we can transfer those logs to an Eric Zimmerman tool called Timeline Explorer.
   ```powershell
    C:\DFIR_Tools\ZimmermanTools\net6\PECmd.exe -q -d C:\Cases\Prefetch\ --csv "C:\Cases\Analysis\ --csvf prefetch.csv"
   ```
<img src="https://i.imgur.com/ypuSCL9.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

This command runs the `PECmd.exe` tool from Eric Zimmerman's suite with the following options:

- `-q`: Enables quiet mode (minimal output).
- `-d C:\Cases\Prefetch\`: Specifies the directory containing Prefetch files to analyze.
- `--csv "C:\Cases\Analysis\ --csvf prefetch.csv"`: Exports the analysis results to a CSV file named `prefetch.csv` in the `C:\Cases\Analysis\` directory.

The tool processes the Prefetch files and saves the results in a CSV format for further analysis.

2. Open Timeline Explorer from the Desktop, and open the two newly created csv files.
<img src="https://i.imgur.com/h99tq4s.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/6eywqw1.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

3. Click on the RunTime column to make the timeline in either ascending or descending order. This helps us in making a proper timeline of events.

4. According to our Scenario, Bill downloaded Burpsuite. So lets simply search for the term in Timeline Explorer. 
- We see only a single hit on the term burpsuite, location being
   ```powershell
      USERS\BILL.LUMBERGH\DOWNLOADS\BURPSUITE-PRO-CRACKED.EXE
   ```

<img src="https://i.imgur.com/UbPl3l9.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

- It's evident Bill downloaded a cracked version of Burpsuite applciation on
  ```powershell
  2024-03-12 at 18:36🕚
  ```
5. Clear the search term of Burpsuite, and now lets analyze the nearby logs. As shown below just before burpsuite was installed, a file called 7ZG.EXE was executed, which is the potential malware.

<img src="https://i.imgur.com/uJCPhzZ.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

6. Now we simply scroll a bit, and see if there are any suspicious looking files, and we come across a few, as shwon below.

<img src="https://i.imgur.com/GysrugE.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/akoJ51H.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/xG9WuO4.png" height="85%" width="85%" alt="Disk Sanitization Steps"/>









