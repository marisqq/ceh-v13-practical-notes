# Lab setup

## Architecture

```
Attacker (Parrot/Kali)
        |
        └── Isolated network (10.10.55.0/24)
                |
                ├── Web Target (10.10.55.10) — DVWA + WordPress + FTP + MySQL
                ├── Linux Target (10.10.55.11) — SSH, SNMP, NFS, SMTP, weak users
                ├── AD Target (10.10.55.22) — Samba AD DC
                └── Steg/Crypto (10.10.55.13) — challenge files
```

---

## Targets

### Web — 10.10.55.10
Apache, MariaDB, DVWA, WordPress, FTP, MySQL

Flags:
1. `http://10.10.55.10/dvwa` — admin/password, SQLi, cmd injection, XSS
2. `http://10.10.55.10/sqli.php?id=1` — SQLMap target
3. `http://10.10.55.10/cmd.php` — command injection
4. `http://10.10.55.10/wordpress` — WPScan target
5. FTP anonymous → flag.txt
6. FTP brute force ftpuser → secret.txt

### Linux — 10.10.55.11
SSH, SNMP, NFS, SMTP, hash files

Flags:
1. SSH brute force: alice → "cupcake"
2. SNMP: `snmpwalk -v2c -c public 10.10.55.11`
3. NFS: `showmount -e 10.10.55.11` → mount → confidential.txt
4. SMTP: `smtp-user-enum -M VRFY -U users.txt -t 10.10.55.11`
5. Hash cracking: NTLM hashes in /home/alice/hashes.txt

### AD — 10.10.55.22
Samba AD DC (Kerberos 88, LDAP 389, SMB 445)

Flags:
1. Identify DC: `nmap -p 88,389,445 10.10.55.22`
2. LDAP: `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM"`
3. AS-REP Roast Joshua
4. Password spray with "cupcake"
5. SMB: `enum4linux -a 10.10.55.22`

---

## Steg + crypto challenges

```bash
mkdir -p ~/ceh-ctf/steg-crypto && cd ~/ceh-ctf/steg-crypto

# steghide (JPEG)
steghide embed -cf sample.jpg -ef secret.txt -p "password123"
# extract: steghide extract -sf sample.jpg -p password123

# snow (whitespace steg)
snow -C -m "FLAG{snow_whitespace_secret}" -p "letmein" cover.txt stego.txt
# extract: snow -C -p "letmein" stego.txt

# binwalk (embedded file)
cat photo.jpg secret.zip > suspicious.jpg
# extract: binwalk -e suspicious.jpg

# hash comparison
md5sum file*.txt

# hash cracking
# hashcat -m 0 crack_this.txt rockyou.txt
```

---

## Wireshark filters

| What | Filter |
|------|--------|
| FTP password | `ftp.request.command == "PASS"` |
| HTTP POST | `http.request.method == "POST"` |
| Traffic from IP | `ip.addr == 10.10.55.10` |
| DNS | `dns` |
| SMTP | `smtp` |
| Find scanner | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| Extract data | Right-click → Follow → TCP Stream |

---

## Flag checklist

### Web (10.10.55.10)
- [ ] SQLMap: find databases via sqli.php
- [ ] SQLMap: dump DVWA users table
- [ ] Command injection: /etc/passwd via cmd.php
- [ ] WPScan: enumerate users
- [ ] WPScan: brute force login
- [ ] Nikto: web server version
- [ ] dirb: hidden directories
- [ ] FTP anonymous: flag.txt
- [ ] FTP brute force: ftpuser's secret.txt

### Linux (10.10.55.11)
- [ ] Hydra SSH: alice's password
- [ ] SNMP: system info
- [ ] NFS: mount share, confidential.txt
- [ ] SMTP: valid users
- [ ] Hash cracking: NTLM from hashes.txt

### AD (10.10.55.22)
- [ ] Nmap: identify DC (88+389)
- [ ] LDAP: domain users
- [ ] AS-REP Roast: Joshua's hash
- [ ] Password spray: who else uses "cupcake"
- [ ] SMB: shares and users

### Steg/Crypto
- [ ] Steghide: extract from JPEG
- [ ] Snow: extract from text
- [ ] binwalk: extract embedded file
- [ ] Hash comparison: find tampered file
- [ ] MD5 cracking
- [ ] VeraCrypt: mount and read

### Wireshark
- [ ] FTP credentials in pcap
- [ ] Scanning activity
- [ ] Follow TCP stream
