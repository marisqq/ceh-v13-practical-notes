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

