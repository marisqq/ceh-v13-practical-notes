# What Goes Where  
  
## Lab Targets  
  
| Target | What You Practice |  
|--------|-------------------|  
| Web Target (10.10.55.10) | DVWA (SQLi, cmd injection, XSS), WordPress (WPScan), FTP (Hydra), MySQL, Nikto, dirb |  
| Linux Target (10.10.55.11) | SSH brute force (Hydra), SNMP enum, NFS shares, SMTP enum, hash files |  
| AD Target (10.10.55.22) | Samba AD DC, Kerberos (port 88), LDAP (389), AS-REP Roast, Kerberoast, SMB enum, password spraying |  
  
Your desktop = attacker machine (Parrot/Kali/Fedora with nmap, hydra, sqlmap, metasploit, etc)
  
---  
  
## Practice on Desktop (no VM needed)  
  
| Topic | How | Tool |  
|-------|-----|------|  
| Wireshark/pcap analysis | Download sample pcaps from wireshark.org/SampleCaptures | Wireshark |  
| Steganography (steghide) | Create test images with embedded data locally | steghide, binwalk, exiftool |  
| Snow (whitespace steg) | Create test files locally | stegsnow |  
| Hash cracking | Create hash files locally, crack with hashcat/john | hashcat, john |  
| Hash identification | Paste hashes into hash-identifier | hash-identifier |  
| File hash comparison | Create files, tamper one, compare md5sums | md5sum, sha256sum |  
  
---  
  
## Practice in EC-Council Labs Only (tool not available locally)  
  
| Topic | Why EC-Council Lab |  
|-------|-------------------|  
| OpenStego | Windows GUI tool, easier in their Windows VM |  
| VeraCrypt | Need to create/mount encrypted volumes on Windows |  
| CrypTool | Windows-only encryption tool |  
| BCTextEncoder | Windows-only |  
| Detect It Easy (DIE) | Malware analysis, need sample PE files |  
| PhoneSploit/ADB | Need Android emulator (port 5555 target) |  
| OpenVAS/GreenBone | Heavy to install, just use their lab once |  
| Zenmap | Windows GUI nmap, trivial to use |  
  
> For these: speed-run the relevant EC-Council lab module once. They're GUI-click tools, 30 min each max.  
  
---  
  
## EC-Council Labs Worth Speed-Running (for tools above)  
  
| Lab | Module | Time | What to learn |  
|-----|--------|------|---------------|  
| Malware Threats | M07 | 30min | Open file in DIE, note entropy + packer |  
| Sniffing | M08 | 30min | Wireshark filters (but you can practice locally too) |  
| Mobile Platforms | M17 | 30min | PhoneSploit → adb connect → pull files |  
| Cryptography | M20 | 30min | VeraCrypt mount, CrypTool decrypt, BCTextEncoder |  
| Vulnerability Analysis | M05 | 30min | OpenVAS scan, find CVSS score |  
  
**Total EC-Council lab time: ~2.5 hours** for the Windows/GUI tools you can't replicate  
  
---  
  
## Practice Schedule  
  
### Days 1-3: Lab CTF grinding  
- Build lab → run through all flags  
- Repeat until you can do each flag in <5 minutes  
- Focus order: Web attacks → Password cracking → AD attacks → Enumeration  
  
### Day 4: EC-Council speed-runs  
- M07 (DIE), M17 (PhoneSploit), M20 (VeraCrypt/CrypTool), M05 (OpenVAS)  
- 30 min each, just learn the GUI workflow  
  
### Day 5: Full mock exam  
- Set 6-hour timer  
- Scan all lab subnets from scratch  
- Try to find every flag using only your cheatsheets  
- Practice time management  
