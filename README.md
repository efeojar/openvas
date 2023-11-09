## AZURE OPENVAS Vulnerability Management
OpenVAS, the Open Vulnerability Assessment System, is a critical open-source tool for identifying and managing security vulnerabilities in computer systems and networks. Its essence lies in comprehensive vulnerability assessment, supported by a continually updated database of known vulnerabilities. With a user-friendly interface and customization options, it enables users to scan hosts or entire networks, generate detailed reports, and integrate with other security systems. OpenVAS facilitates continuous monitoring, ensuring organizations can proactively address vulnerabilities and maintain a robust security posture. Its open-source nature and strong community support make it accessible and invaluable for organizations of all sizes in safeguarding their digital assets. In this project, I will demonstrate how to use OPENVAS to scan through a vulnerable VM that is running outdated software and how to remediate it.
# Prerequisites
* Computer with the Internet
* Azure Account - May be Possible to do this with a Free Subscription
Create a Free Azure Account

 [Sign up Azure](https://azure.microsoft.com/en-us/free/)
# OPENVAS Vulnerability Scanner

<img width="1291" alt="Screenshot 2023-11-07 at 6 04 11 PM" src="https://github.com/efeojar/openvas/assets/66268247/f1ac7aa2-4167-4559-860c-a20aa5d11733">

1. First, log in to the Azure Portal
2. Find "OpenVAS secured and supported by HOSSTED": In the Azure Portal, use the search bar located in the top navigation and search for "OpenVAS secured and supported by HOSSTED." Locate and select the appropriate solution available in the Azure Marketplace.
3. Select Configuration: Opt for the "Start with a predefined configuration" option. To align with your request, choose the configuration labeled as the least secure.
4. VM Configuration: Provide the necessary information as follows:
* Resource Group: Create a new group "Vulnerability-Management."
* VM Name: Assign a name to your VM, e.g., "OpenVAS."
* Region: Pick a region, like "East US 2."
* Virtual Network (Vnet): Select an existing Vnet or create a new one if needed.
* Authentication: Set the username and password
* Diagnostics: Ensure that Boot Diagnostic is disabled. Due to resource conservation, security, and simplicity boot disabled is needed. 
5. Create the VM: Review your settings and, when ready, initiate the VM deployment by clicking the "Create" button.
6. SSH Access to the VM: Once the VM creation process is complete, establish an SSH connection to the VM using an appropriate terminal application. On Windows, you can utilize PowerShell with SSH support, while macOS and Linux users can employ the Terminal. Use the provided credentials for access. Be patient during the OpenVAS deployment process.
<img width="618" alt="Screenshot 2023-11-07 at 6 29 26 PM" src="https://github.com/efeojar/openvas/assets/66268247/c27b62f8-81f3-4958-8602-2b509251b6c9">

# Create a Client VM and Make it Vulnerable
1. Search for Virtual Machines and create a new Virtual Machine.
2. Configure the VM:
* Resource Group: (Make sure it's added in the same resources as the OPENVAS)
* VM Name: Win10-Vulnerable
* Region: Same as the OpenVAS VM (East US 2)
* Virtual Network: Same as OpenVAS
* Image: Windows 10 Pro
* Size: Any size with 2 vCPUs
* Username: create a username and password
* Networking: Same Vnet as OpenVAS
3. Create the VM.
4. Once the VM is created, copy the public IP and RDP into the machine.
5. After logging in, make the VM vulnerable:
* Disable the Windows Firewall
* Gather up some Old Software from Google 
* Install an Old Version of FireFox: Firefox Setup 97.0b5
* Install an Old Version of VLC Player: vlc-1.1.7-win32
* Install an Old Version of Adobe Reader: 10.0_AdbeRdr1000_en_US_1_
* Restart the VM.

# Configure OpenVAS to Perform the First Unauthenticated Scan against our Vulnerable VM
<img width="1657" alt="Screenshot 2023-11-07 at 10 14 00 PM" src="https://github.com/efeojar/openvas/assets/66268247/701862a3-02ef-4e25-91a6-0d6fe67cf648">

1. Login to OpenVAS with the login details and URL given when you SSH into OpenVasb and navigate to Assets > Hosts > New Host.
2. Add the Client VM PRIVATE IP Address to the host.
3. Create a New Target from the Host, name it whatever you like.
4. Take note of the credentials. We will add SMB credentials later.
5. Create a new Task:
* Name your task and comment
* Scan Targets: Name your Target
6. Save the Task.
7. Start the "Scan - Task name created".
8. Once the scan is finished, click the date under "Last Report" to see the results.
9. Take note of the Tabs, especially the "Results" tab.

# Make Configurations for Credentialed Scans (Within VM)
1. Disable Windows Firewall.
2. Disable User Account Control.
3. Enable Remote Registry.
4. Set Registry Key:
* Launch Registry Editor (regedit.exe) in "Run as administrator" mode.
* Navigate to HKEY_LOCAL_MACHINE hive.
* Open SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System key.
* Create a new DWORD (32-bit) value with the following properties:
  * Name: LocalAccountTokenFilterPolicy
  * Value: 1
* Close Registry Editor.
5. Restart the VM.
Just so you know, in a Windows credential scan, these steps are taken to enhance scan efficiency and access to the target system. Disabling Windows Firewall allows the scanning tool to communicate without port restrictions. Disabling User Account Control (UAC) prevents pop-up interruptions and enabling Remote Registry access is crucial for gathering system information. The "LocalAccountTokenFilterPolicy" registry key modification allows the scanning tool to access the target system's registry. These actions are temporary and specific to scanning tasks, as they might reduce security for normal system operation.

# Make Configurations for Credentialed Scans (OpenVAS)
1. Go to Configuration > Credentials > New Credential.
2. Name / Comment: Create a name.
3. Allow Insecure Use: Yes.
4. Username: VM username created
5. Password: VM Password
* Save.
6. Go to Configuration > Targets > CLONE the Target we made before.
7. NEW Name / Comment: Create a name.
8. Ensure the Private IP is still accurate.
9. Credentials > SMB > Select the Credentials we just made: Azure VM Credentials.
* Save.

# Execute Credentialed Scan against our Vulnerable Windows VM

<img width="1649" alt="Screenshot 2023-11-08 at 8 35 35 AM" src="https://github.com/efeojar/openvas/assets/66268247/d23b51c7-4392-49bc-b4b4-1fdd3a1ff53c">
<img width="1653" alt="Screenshot 2023-11-08 at 8 45 49 AM" src="https://github.com/efeojar/openvas/assets/66268247/ce6cfe30-91e4-47a7-a1a8-3cc2698ff69c">
<img width="1655" alt="Screenshot 2023-11-08 at 8 46 48 AM" src="https://github.com/efeojar/openvas/assets/66268247/7d70ae70-a7ba-455b-8190-a5387f5f8de7">

1. Within Greenbone / OpenVAS, go to Scans > Tasks.
2. CLONE the "Scan - Create a new name to separate them -  Task and Edit it.
3. Name / Comment: "Scan - Use the same name as #2 - Credentialed".
4. Targets: The same target was created earlier - Credentialed Scan.
* Save.
5. Click the Play button to launch the new Credentialed Scan and wait for it to finish.

# Remediate Vulnerabilities
1. Log back into your Vulnerable VM.
2. Uninstall Adobe Reader, VLC Player, and Firefox or Upgrade to the latest version. 
Restart the VM.
3. Re-initiate the "Credentialed" scan and observe the results.

# Verify Remediations
<img width="1652" alt="Screenshot 2023-11-08 at 9 27 53 AM" src="https://github.com/efeojar/openvas/assets/66268247/6b2d6176-6b25-43b5-b264-a20982a3f147">
