
Microsoft Disclaimer for custom scripts
# ================================================================================================================
# The sample files are not supported under any Microsoft standard support program or service. The sample files
# are provided AS IS without warranty of any kind. Microsoft further disclaims all implied warranties including, 
# without limitation, any implied warranties of merchantability or of fitness for a particular purpose. The entire
# risk arising out of the use or performance of the sample files and documentation remains with you. In no event
# shall Microsoft, its authors, or anyone else involved in the creation, production, or delivery of the files be
# liable for any damages whatsoever (including, without limitation, damages for loss of business profits, business
# interruption, loss of business information, or other pecuniary loss) arising out of the use of or inability to 
# use the sample scripts or documentation, even if Microsoft has been advised of the possibility of such damages.
# ================================================================================================================



# Migrate2MDAV/MDE (Windows 10 and Server 2019)
__Migrating from 3rd party AV to Microsoft Defender AV using SCCM/MECM__

Migrating from 3rd Party AV to Microsoft Defender Anti-Virus (MDAV) and Onboarding to Microsoft Defender for Endpoint (MDE) can be a complex process depending on the AV vendor, organizational policies, infrastructure, etc. However, the following is a process I have developed through trial and error. The AV-Migration-MDE-TS zipped file is a Task Sequence template for SCCM/MECM. It is assumed the user of this file understands how to build/run a Task Sequence, Packages, etc. You must edit and configure the Tasks to meet your organization's requirements. This is the order of actions I have found to work best, you may find a different variation that works for you.

**Migration Steps:**

1. __Plan MDE RBAC and Device Groups__ (https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/user-roles?view=o365-worldwide)

	A.) Download "MDE Roles and Groups" spreadsheet. Planning this first will save you time later.

2. __On a single Test device export your current AV Firewall policies:__ (https://github.com/JesseEsquivel/MDATP/tree/master/Scripts/Migrations)- *Note, this script may not work in all cases due to the various ways organizations create Firewall policies. The script parses an xml file and looks for certain parameters that some organizations omit or modify.*

 	 A.) Export SEP firewall policies to CSV
	 
	 B.) Import csv file into running Windows Firewall using PowerShell Script (https://winaero.com/export-and-import-specific-firewall-rule-in-windows-10/)
	 
	 C.) On same device Export policies to Folder- this is saved as a .wfw which can then be imported via SCCM/GPO in Step 6 below (testpolicy.wfw)
	 
	
3. __Configure GPO's (if devices are not allowed to connect to Internet directly)__(https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/production-deployment?view=o365-worldwide)

    A.) Administrative Templates > Windows Components > Data Collection and Preview Builds > Configure Authenticated Proxy usage for the Connected User Experience and Telemetry Service; Set it to Enabled and select Disable Authenticated Proxy usage

    B.) Administrative Templates > Windows Components > Data Collection and Preview Builds > Configure connected user experiences and telemetry.
		Set to Enabled
		Enter Proxy Server name

    C.) Tag devices: (per "MDE Roles and Groups" spreadsheet)
    
	Use the following registry key entry to add a tag on a device:
	
		Registry key: HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection\DeviceTagging\
	
		Registry key value (REG_SZ): Group
	
		Registry key data: Name of the tag you want to set

4. __Onboard devices via SCCM MDATP Policy (Windows 10 and Server 2019 only using the .onboarding file from the MDE Portal)__(https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/onboarding-endpoint-configuration-manager?view=o365-worldwide)
   
5. __Configure Antimalware Policies in SCCM__

6. __Configure and run Task Sequence:__

	A.) Uninstall 3rd party AV (See notes below)
	
	B.) Enable Windows Firewall: netsh advfirewall set allprofiles state on
	
	C.) Import Firewall Policies: Netsh advfirewall import c:\temp\testpolicy.wfw (if applicable)
	
	D.) update MDAV definitions
	
	E.) Reboot (this will remove remnants of 3rd party AV and put MDAV in Active mode upon reboot)
	
	F.) Run MDE Detection Test:
		powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden $ErrorActionPreference= 'silentlycontinue';(New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-WDATP-test\\invoice.exe');Start-Process 'C:\\test-WDATP-test\\invoice.exe'
		



__**Notes- Uninstalling Vendor AV:__**

**Symantec**: https://knowledge.broadcom.com/external/article/151297/uninstall-the-endpoint-protection-client.html

**McAfee**: https://kc.mcafee.com/corporate/index?page=content&id=KB90895

**CrowdStrike Falcon**: https://help.redcanary.com/hc/en-us/articles/360052302854-Installing-and-uninstalling-the-Crowdstrike-Falcon-sensor-on-Windows

**TrendMicro**: https://success.trendmicro.com/global-search?keyword=Uninstall+Trend+Micro



