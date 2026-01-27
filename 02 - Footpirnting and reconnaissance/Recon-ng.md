help  

marketplace install all - install all available modules  

modules search - list all modules  

workspaces - list available workspaces  

wokrspaces create CEH - create workspace  

db insert domains - issue domain name to work on  
![](../img/Pasted%20image%2020260122212733.png)  

module brute - to list all bruteforce related modules  
load recon/domains/-hosts/brute_hosts - to load module  

![](../img/Pasted%20image%2020260122213201.png)  

back - move back to workspace/exit module  

to resolve hosts using bing - modules load recon/domains-hosts/bing_domain_web  

modules load reverse_resolve - list resolve module  

load recon/hosts-hosts/reverse_resolve - start reverse lookup  
run  

modules load reporting - report generation modules  
modules load reporting/html  
options set FILENAME /home/attacker/Desktop/results.html  
options set CREATOR - your name  
options set CUSTOMER - who was report for  


Gather personnel information  
--  

recon-ng  
modules load recon/domains/contacts/whois_pocs - uses ARIN whois to harvest data from whois queries get person names  

info command - view options for using modules  

options set SOURCE facebook.com - select target  
![](../img/Pasted%20image%2020260122215533.png)  

extract list of subdomains and IP addresses   
--  
recon-ng   
modules load recon/domains-hosts/hackertarget  
options set SOURCE xyz.com  