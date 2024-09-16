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
1. Our goal is to look at the malicious executables ran, which directories they accesses, how many times they were ran, and a bit more.
