# Study plan

## Lab targets

| Target | What to practice |
|--------|-----------------|
| Web (10.10.55.10) | DVWA (SQLi, cmd injection, XSS), WordPress (WPScan), FTP (Hydra), MySQL, Nikto, dirb |
| Linux (10.10.55.11) | SSH brute force (Hydra), SNMP enum, NFS shares, SMTP enum, hash files |
| AD (10.10.55.22) | Samba AD DC, Kerberos (88), LDAP (389), AS-REP Roast, Kerberoast, SMB enum, password spraying |

Desktop = attacker machine (Parrot/Kali/Fedora)

---

## Desktop practice (no VM needed)

| Topic | How | Tool |
|-------|-----|------|
| Wireshark/pcap | Sample pcaps from wireshark.org/SampleCaptures | Wireshark |
| Steganography | Create test images with embedded data | steghide, binwalk, exiftool |
| Snow (whitespace steg) | Create test files locally | stegsnow |
| Hash cracking | Create hash files, crack with hashcat/john | hashcat, john |
| Hash identification | Paste hashes into hash-identifier | hash-identifier |
| File hash comparison | Create files, tamper one, compare md5sums | md5sum, sha256sum |

---

## EC-Council labs only (tools not available locally)

| Topic | Why |
|-------|-----|
| OpenStego | Windows GUI, easier in their VM |
| VeraCrypt | Need encrypted volumes on Windows |
| CrypTool | Windows-only |
| BCTextEncoder | Windows-only |
| DIE | Malware analysis, need sample PE files |
| PhoneSploit/ADB | Need Android emulator (port 5555) |
| OpenVAS | Heavy to install, just use their lab |
| Zenmap | Windows GUI nmap, trivial |

Speed-run each module once, 30 min max. GUI-click tools.

---

## EC-Council labs worth doing

| Lab | Module | Time | What to learn |
|-----|--------|------|---------------|
| Malware Threats | M07 | 30min | DIE — entropy + packer |
| Sniffing | M08 | 30min | Wireshark filters |
| Mobile Platforms | M17 | 30min | PhoneSploit → adb connect → pull files |
| Cryptography | M20 | 30min | VeraCrypt mount, CrypTool, BCTextEncoder |
| Vulnerability Analysis | M05 | 30min | OpenVAS scan, CVSS score |

Total: ~2.5 hours for the Windows/GUI stuff

---

## Schedule

### Days 1-3: Lab grinding
- Run through all flags
- Repeat until each flag takes <5 min
- Focus: web attacks → password cracking → AD → enumeration

### Day 4: EC-Council speed-runs
- M07 (DIE), M17 (PhoneSploit), M20 (VeraCrypt/CrypTool), M05 (OpenVAS)
- 30 min each, just learn the GUI flow

### Day 5: Mock exam
- 6 hour timer
- Scan lab from scratch
- Only use cheatsheets
- Practice time management
