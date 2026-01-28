Scan types in this section:  
Packet fragmentation, source routing, source port manipulation, IP address decoy, IP address spoofing, creating custom packets, randomizing host order, sending Bad checksum.  

Split IP packet in fragments to avoid firewall  

nmap -f [ip]  

![](../img/Pasted%20image%2020260128204246.png)  

------------------------------------------------------------

Change source port to avoid firewall (If only allows well-known ports)  

nmap -g 80 or --source-port   

![](../img/Pasted%20image%2020260128204933.png)  

Wireshark:  

![](../img/Pasted%20image%2020260128205043.png)  

-------------------------------------------


Scan using smaller MTU  

nmap -mtu 8 [ip]  


![](../img/Pasted%20image%2020260128205506.png)  


---------------------------

IP address decoy - makes hard for IDS to detect which IP is scanning  

nmap -D RND:10 10.10.1.11  

![](../img/Pasted%20image%2020260128210535.png)  



-----------------------------




