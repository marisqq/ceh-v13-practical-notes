TCP scan open ports (three way handshake)  
  
nmap -sT -v [ip]  
  
---------------------------------------  
  
Service detection  
  
nmap -sV -v [ip]  
  
Checks running services, version number = exploit  
  
-----------------------------------------------  
  
Aggressive scan  
  
nmap -A 10.10.1.* - scan subnet or IP range  
  
  
Agressive scan supports - -O (OS detection) -sV, -sC and --traceroute  
  
-------------------------------  
  
Port scan if firewall is enabled  
---------------------------------------------  
  
Stealth scan - half open to bypass  
nmap -sS -v 10.10.1.22  
  
------------------------------------------  
  
xmas scan - send TCP frame with FIN, URG andPUSH flags. If port closed it will send RST  
  
nmap -sX -v [ip]  
  
-------------------------------  
  
TCP maimon scan, FIN/ACK, chec if port is Open|Filtered  
  
nmap -sM -v [ip]  
  
----------------------------------------------------------  
TCP ACK scan - understand if port is filtered or unfiltered (stateful firewall is present)  
  
nmap -sA -v [ip]  
  
ACK flag probe, no response implies that port is filtered and RST means port is not filtered  
  
------------------------------  
UDP scan to check UDP services like DNS 53, DHCP, TFTP etc.  
  
nmap -sU [ip]  
  
--------------------------------------  
  
  
IDLE/IPID scan with spoofed source address  
  
nmap -sl -v [ip]  
  
------------------  
SCPT INIT  
  
nmap -sY -v  
  
----------------  
SCTP COOKIE ECHO Scan  
  
nmap -sZ -v [ip]  
  
  
## Scan Type Quick Reference for Exam  
  
| Scan | Flag | When to use | Key detail |  
|------|------|-------------|------------|  
| TCP Connect | `-sT` | Full 3-way handshake, logged | Default for non-root |  
| SYN Stealth | `-sS` | Half-open, stealthier | Default for root |  
| FIN | `-sF` | Only FIN flag set | Open ports don't respond |  
| Xmas | `-sX` | FIN+URG+PSH flags | Open ports don't respond |  
| NULL | `-sN` | No flags set | Open ports don't respond |  
| ACK | `-sA` | Map firewall rules | Shows filtered vs unfiltered |  
| UDP | `-sU` | DNS, SNMP, DHCP | Slow, use with specific ports |  
| IDLE | `-sI` | Completely blind scan | Uses zombie host |  
| Maimon | `-sM` | FIN/ACK probe | BSD-derived systems |  
| SCTP INIT | `-sY` | Like SYN but for SCTP | VoIP, telecom |  
  
## Hping3 - Packet Crafting  
`hping3 -S [ip] -p 80 -c 5` - SYN scan port 80, 5 packets  
`hping3 -A [ip] -p 80` - ACK scan  
`hping3 -F -P -U [ip] -p 80` - Xmas scan (FIN+PUSH+URG)  
`hping3 --udp [ip] -p 53` - UDP scan  
`hping3 -1 [ip]` - ICMP ping  
`hping3 -S [ip] -p 80 --flood` - SYN flood (DoS)  
`hping3 -S --scan 1-1000 [ip]` - scan port range  
  
## Important Nmap Output Formats  
`nmap -oN output.txt` - normal text  
`nmap -oX output.xml` - XML (used by Metasploit)  
`nmap -oG output.gnmap` - grepable  
`nmap -oA output` - all three formats  
  
## Nmap Scripting Engine (NSE)  
`nmap --script vuln [ip]` - run all vuln scripts  
`nmap --script=http-enum [ip]` - enumerate web directories  
`nmap --script=smb-vuln* [ip]` - check SMB vulnerabilities  
`nmap -sC [ip]` - default scripts (same as --script=default)  
  
## Common Port Numbers for Exam  
| Port | Service |  
|------|---------|  
| 21 | FTP |  
| 22 | SSH |  
| 23 | Telnet |  
| 25 | SMTP |  
| 53 | DNS |  
| 67/68 | DHCP |  
| 69 | TFTP |  
| 80 | HTTP |  
| 110 | POP3 |  
| 135 | MSRPC |  
| 137-139 | NetBIOS |  
| 143 | IMAP |  
| 161/162 | SNMP |  
| 389 | LDAP |  
| 443 | HTTPS |  
| 445 | SMB |  
| 993 | IMAPS |  
| 995 | POP3S |  
| 1433 | MSSQL |  
| 1521 | Oracle |  
| 2049 | NFS |  
| 3306 | MySQL |  
| 3389 | RDP |  
| 5432 | PostgreSQL |  
| 5900 | VNC |  
| 8080 | HTTP Proxy |  
