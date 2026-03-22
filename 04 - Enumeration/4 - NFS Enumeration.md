![](../img/Pasted%20image%2020260129214309.png)  
  
Add roles and features  
  
![](../img/Pasted%20image%2020260129220133.png)  
  
--------------------------------------  
  
Check if nfs is open  
  
![](../img/Pasted%20image%2020260129220938.png)  
  
`nmap -p 2049`[ip] | Scan if NFS is open  
  
-----------------------------------------------  
  
SuperEnum  
-------------  
  
![](../img/Pasted%20image%2020260312104416.png)  
  
-Sucks, cause no option to clearly see what was canned, givees errors and doesnt stop on its own  
  
-----------------------------------------------  
  
## NFS Enumeration Commands  
  
`showmount -e [ip]` - show exported shares (most important command)  
  
Mount NFS share:  
`mkdir /tmp/nfs && mount -t nfs [ip]:/share /tmp/nfs`  
  
## Nmap NFS Scripts  
`nmap -p 2049 --script nfs-showmount [ip]` - show NFS exports  
`nmap -p 2049 --script nfs-ls [ip]` - list NFS directories  
`nmap -p 2049 --script nfs-statfs [ip]` - disk stats  
  
-----------------------------------------------  
  
## RPCinfo  
`rpcinfo -p [ip]` - list RPC services (NFS runs over RPC)  
