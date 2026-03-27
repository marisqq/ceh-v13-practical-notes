# Lab setup

## Architecture

```
Attacker Machine (Parrot/Kali)
        |
        └── Isolated Lab Network (10.10.55.0/24)
                |
                ├── Web Target (10.10.55.10) — DVWA + WordPress + FTP + MySQL
                ├── Linux Target (10.10.55.11) — SSH, SNMP, NFS, SMTP, weak users
                ├── AD Target (10.10.55.22) — Samba AD DC
                └── Steg/Crypto (10.10.55.13) — file server with challenges
```

---

## Lab Targets Overview

### Web Target — 10.10.55.10
Services: Apache, MariaDB, DVWA, WordPress, FTP, MySQL

**CTF Flags:**
1. `http://10.10.55.10/dvwa` — login admin/password, practice SQLi, command injection, XSS
2. `http://10.10.55.10/sqli.php?id=1` — SQLMap target
3. `http://10.10.55.10/cmd.php` — command injection
4. `http://10.10.55.10/wordpress` — WPScan target
5. FTP anonymous → flag.txt
6. FTP brute force ftpuser → secret.txt

### Linux Target — 10.10.55.11
Services: SSH, SNMP, NFS, SMTP, hash files

**CTF Flags:**
1. SSH brute force: alice → "cupcake"
2. SNMP enum: `snmpwalk -v2c -c public 10.10.55.11`
3. NFS: `showmount -e 10.10.55.11` → mount → read confidential.txt
4. SMTP: `smtp-user-enum -M VRFY -U users.txt -t 10.10.55.11`
5. Hash cracking: NTLM hashes in /home/alice/hashes.txt

### AD Target — 10.10.55.22
Services: Samba AD DC (Kerberos 88, LDAP 389, SMB 445)

**CTF Flags:**
1. Identify DC: `nmap -p 88,389,445 10.10.55.22`
2. LDAP enum: `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM"`
3. AS-REP Roast Joshua
4. Password spray with "cupcake"
5. SMB enum: `enum4linux -a 10.10.55.22`

---

## Steg + Crypto Challenge Files

Create challenge files on your desktop:

```bash
mkdir -p ~/ceh-ctf/steg-crypto && cd ~/ceh-ctf/steg-crypto

# --- Steganography ---
# 1. Steghide (JPEG)
steghide embed -cf sample.jpg -ef secret.txt -p "password123"
# Extract: steghide extract -sf sample.jpg -p password123

# 2. Snow (whitespace steganography)
snow -C -m "FLAG{snow_whitespace_secret}" -p "letmein" cover.txt stego.txt
# Extract: snow -C -p "letmein" stego.txt

# 3. Binwalk (embedded file)
cat photo.jpg secret.zip > suspicious.jpg
# Extract: binwalk -e suspicious.jpg

# --- Crypto ---
# 4. Hash comparison (find tampered file)
md5sum file*.txt

# 5. Hash cracking
# hashcat -m 0 crack_this.txt rockyou.txt
```

---

## Wireshark Practice

**Filters to memorize:**

| Find This | Filter |
|-----------|--------|
| FTP password | `ftp.request.command == "PASS"` |
| HTTP POST (login forms) | `http.request.method == "POST"` |
| All traffic from IP | `ip.addr == 10.10.55.10` |
| DNS queries | `dns` |
| SMTP traffic | `smtp` |
| Find the scanner | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| Extract file/data | Right-click → Follow → TCP Stream |

---

## CTF Flag Checklist

### Web (10.10.55.10)
- [ ] SQLMap: find database names via sqli.php
- [ ] SQLMap: dump DVWA users table
- [ ] Command injection: read /etc/passwd via cmd.php
- [ ] WPScan: enumerate WordPress users
- [ ] WPScan: brute force WordPress login
- [ ] Nikto: identify web server version
- [ ] dirb: find hidden directories
- [ ] FTP anonymous: retrieve flag.txt
- [ ] FTP brute force: get ftpuser's secret.txt

### Linux (10.10.55.11)
- [ ] Hydra SSH: crack alice's password
- [ ] SNMP: enumerate system info with snmpwalk
- [ ] NFS: mount share, read confidential.txt
- [ ] SMTP: enumerate valid users
- [ ] Hash cracking: crack NTLM hashes from hashes.txt

### AD (10.10.55.22)
- [ ] Nmap: identify as Domain Controller (port 88+389)
- [ ] LDAP: enumerate domain users
- [ ] AS-REP Roast: get Joshua's hash and crack it
- [ ] Password spray: find other users with "cupcake"
- [ ] SMB: enumerate shares and users

### Steg/Crypto (local files)
- [ ] Steghide: extract from JPEG
- [ ] Snow: extract from text file
- [ ] binwalk: extract embedded file
- [ ] Hash comparison: find tampered file
- [ ] MD5 cracking
- [ ] VeraCrypt: mount and read contents

### Wireshark (local pcap)
- [ ] Find FTP credentials in pcap
- [ ] Identify scanning activity
- [ ] Follow TCP stream to extract data
