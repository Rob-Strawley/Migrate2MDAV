# Migrate2MDAV
__Migrating from 3rd party AV to Microsoft Defender for AV using SCCM/MECM__

Migrating from 3rd Party AV to Microsoft Defender (MDAV) and Onboarding to Microsoft Defender for Endpoint (MDE) can be a complex process depending on the AV vendor, organizational policies, infrastructure, etc. However, the following is a process I have developed through trial and error. The AV-Migration-MDE-TS zipped file is a Task Sequence template for SCCM/MECM. It is assumed the user of this file understands how to build a Task Sequence, Packages, etc. You must edit and configure the Tasks to meet your organization's requirements. This is the order of actions I have found to work best, you may find a variation that works for you.

**Migration Steps:**
1. __On a single Test device export your current AV Firewall policies:__ (https://github.com/JesseEsquivel/MDATP/tree/master/Scripts/Migrations)

 	 	A.) Export SEP firewall policies to CSV

 	 	B.) Import csv file into running Windows Firewall using PowerShell Script (https://winaero.com/export-and-import-specific-firewall-rule-in-windows-10/)

	 	C.) On same device Export policies to Folder- this is saved as a .wfw which can then be imported via SCCM/GPO (testpolicy.wfw)
	 
  	 	D.) Task Sequence command- import testpolicy.wfw

2. __Configure GPO's (if devices are not allowed to connect to Internet directly)__

    A.) Administrative Templates > Windows Components > Data Collection and Preview Builds > Configure Authenticated Proxy usage for the Connected User Experience and Telemetry Service; Set it to Enabled and select Disable Authenticated Proxy usage

    B.) Administrative Templates > Windows Components > Data Collection and Preview Builds > Configure connected user experiences and telemetry.
	    i.Set to Enabled
	    ii.Enter Proxy Server name

    C.) Tag devices: (doesn't necessarily need to be done here but it makes it easier later)
	Use the following registry key entry to add a tag on a device:
	Registry key: HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection\DeviceTagging\
	Registry key value (REG_SZ): Group
	Registry key data: Name of the tag you want to set

3. __Onboard devices via SCCM MDATP Policy (Windows 10 and Server 2019 only using the .onboarding file from the MDE Portal)__
    A. For downlevel Operating Systems configure and use the "MMA Onboard to MDE" Task in the Task Sequence

4. Configure Antimalware Policies in SCCM

5. __Configure and run Task Sequence:__
   	 A. Uninstall 3rd party AV (provided zipped file contains removal of Symantec Endpoint Protection command)
	 B. Enable Windows Firewall: netsh advfirewall set allprofiles state on
	 
    	 C. Import Firewall Policies: Netsh advfirewall import c:\temp\testpolicy.wfw (if applicable)
	 
   	 D. update MDAV definitions
	 
   	 E. Reboot (this will remove remnants of 3rd party AV and put MDAV in Active mode upon reboot)
	 
   	 F. Run MDE Detection Test:
		powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden $ErrorActionPreference= 'silentlycontinue';(New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-WDATP-test\\invoice.exe');Start-Process 'C:\\test-WDATP-test\\invoice.exe'
