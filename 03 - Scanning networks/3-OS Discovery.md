
Active/Passive banner grabbing  
-------------------------------------------

TTL values and TCP windows size to determine OS:  

| Operating System | Default TTL | TCP Window Size |
|------------------|-------------|-----------------|
| Linux            | 64          | 5840            |
| FreeBSD          | 64          | 65535           |
| OpenBSD          | 255         | 16384           |
| Windows          | 128         | 65535 → 1 GB    |
| Cisco Routers    | 255         | 4128            |
| Solaris          | 255         | 8760            |
| AIX              | 255         | 16384           |

Getting OS info with nmap:  
-------------------------------------

nmap -A - takes a lot of time  


![](../img/Pasted%20image%2020260128200848.png)  

------------------------------------------------------------

nmap -O - OS discovery  

![](../img/Pasted%20image%2020260128201021.png)  


--------------------------------------
nmap --script smb-os-dicovery.nse - OS discovery script  

![](../img/Pasted%20image%2020260128201158.png)  


--------------------------------------------------------------

