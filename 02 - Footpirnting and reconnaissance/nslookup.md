nslookup  
	set type=a - configure query, get ip of current domain (local catche)  
	set type=CNAME - get authoritative (primary) name server (real hosting)![](../img/Pasted%20image%2020260122210337.png)  
  
kloth.net/services/nslookup.php  
![](../img/Pasted%20image%2020260122210316.png)  
  
## Common DNS Record Types for Exam  
| Record | Purpose |  
|--------|---------|  
| A | Maps hostname to IPv4 |  
| AAAA | Maps hostname to IPv6 |  
| CNAME | Alias for another domain |  
| MX | Mail exchange servers |  
| NS | Authoritative name servers |  
| SOA | Start of authority (zone info) |  
| PTR | Reverse DNS (IP to hostname) |  
| TXT | SPF, DKIM, verification strings |  
| SRV | Service location records |  
  
## nslookup examples  
```  
nslookup  
set type=mx  
targetdomain.com         → mail servers  
set type=ns  
targetdomain.com         → name servers  
set type=soa  
targetdomain.com         → zone authority info  
```  
  
## dig (Linux alternative)  
`dig targetdomain.com ANY` - all records  
`dig targetdomain.com mx` - mail servers  
`dig axfr @ns1.targetdomain.com targetdomain.com` - attempt zone transfer  
  
## dnsrecon  
`dnsrecon -d targetdomain.com` - standard enum  
`dnsrecon -d targetdomain.com -t axfr` - zone transfer attempt  
  
## Zone Transfer Attack  
If DNS misconfigured, attacker gets full DNS zone (all records):  
`nslookup → server ns1.target.com → set type=any → ls -d target.com`  
`dig axfr @ns1.target.com target.com`  
`dnsrecon -d target.com -t axfr`  
