## DNS Enumeration

### nslookup
```
nslookup  
set type=any  
server [target DNS server]  
ls -d [target domain]       → zone transfer attempt  
```

### dig
`dig @[dns-server] [domain] axfr` - zone transfer  
`dig [domain] any` - all records  

### dnsrecon
`dnsrecon -d [domain]` - standard enumeration  
`dnsrecon -d [domain] -t axfr` - zone transfer  
`dnsrecon -d [domain] -t brt -D /usr/share/wordlists/dnsmap.txt` - brute force subdomains  

### dnsenum
`dnsenum [domain]` - auto zone transfer + brute force + reverse lookup  
`dnsenum --enum [domain]` - full enumeration  

### Nmap DNS Scripts
`nmap --script dns-brute [domain]` - brute force subdomains  
`nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=[domain] -p 53 [dns-server]`  
