


`snpwalk -v1 -c public [ip]` | -v specifies SNP version (1 2c or 3), -c sets community string  

![](../img/Pasted%20image%2020260129210023.png)  

Displays OS, hardware info like CPU, memory, interfaces, counters  

--------------------------------------------------------------

`snpwalk -v2c -c public [ip]` | Displays data transmitted from SNMP agent to SNMP server - including user credentials  

![](../img/Pasted%20image%2020260129210819.png)  

----------------------------------------------

Useful grep functions:   

`snmpwalk -v1 -c public 10.10.1.22 | grep -i "windows\|linux\|hardware"`  

![](../img/Pasted%20image%2020260129211245.png)  

`snmpwalk -v1 -c public 10.10.1.22 | grep -i "sysname"` | reboot info  

![](../img/Pasted%20image%2020260129211513.png)`  
`snmpwalk -v1 -c public 10.10.1.22 | grep STRING ` | get readable strings  


![](../img/Pasted%20image%2020260129211844.png)`  

Decode HEX strings automatically  

`echo "4D 69 63 72 6F 73 6F 66 74 20 48 79 70 65 72 2D 56" | xxd -r -p`  

Pipe and decode all HEX strings for output   
snmpwalk -v2c -c public 10.10.1.22 \  
| grep "Hex-STRING" \
| sed 's/.*Hex-STRING: //' \
| tr -d ' ' \
| xxd -r -p


