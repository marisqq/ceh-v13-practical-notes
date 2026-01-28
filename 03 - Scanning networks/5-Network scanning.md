
Metasploit  
---------------

Discover hosts  

msfconsole  
nmap -Pn -sS -oX Test [ip] - scan whole subnet  

![](../img/Pasted%20image%2020260128211358.png)  

search portscan - to open port scanning modules  

![](../img/Pasted%20image%2020260128211752.png)  

load auxiliary/scanner/portscan/syn - perform syn scan  

set INTERFACE eth0  
set PORTS 80  
set RHOSTS 10.10.1.5-23  
set THREADS 50  

run  

Shows number of active hosts with port 80 open:  

![](../img/Pasted%20image%2020260128212341.png)  

-------------------------------------

Display all TCP ports in target IP addr.  

load auxiliary/scanner/portscan/tcp  

set RHOSTS [target ip]  
run  

![](../img/Pasted%20image%2020260128213109.png)  

----------------------------------
Scan specific port 445 to determine SMB version  

use auxiliary/scanner/smb/smb_version  
set RHOSTS 10.10.1.5-23  
set THREADS 11  

run  

![](../img/Pasted%20image%2020260128213652.png)  

