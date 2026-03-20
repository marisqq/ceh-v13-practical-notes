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

MAC address spoofing  

nmap --spoof-mac 0 [ip] - random MAC  
nmap --spoof-mac Dell [ip] - spoof as Dell vendor  

-----------------------------

Append random data to packets  

nmap --data-length 25 [ip] - add 25 random bytes  

-----------------------------

Randomize host order (avoid pattern detection)  

nmap --randomize-hosts 10.10.1.1-254  

-----------------------------

Send bad checksums (test IDS/firewall)  

nmap --badsum [ip] - no response = firewall/IDS filtering  

-----------------------------

## Evasion Summary Table

| Technique | Flag | Purpose |
|-----------|------|---------|
| Fragment packets | `-f` | Split into 8-byte fragments |
| Custom MTU | `--mtu 8` | Fragment at custom size |
| Decoys | `-D RND:10` | Hide among fake source IPs |
| Source port | `-g 80` | Bypass port-based filtering |
| MAC spoof | `--spoof-mac 0` | Change MAC address |
| Data padding | `--data-length 25` | Evade signature detection |
| Randomize hosts | `--randomize-hosts` | Avoid sequential scan pattern |
| Bad checksum | `--badsum` | Test if IDS validates checksums |
| Idle scan | `-sI zombie` | Use zombie for complete stealth |
