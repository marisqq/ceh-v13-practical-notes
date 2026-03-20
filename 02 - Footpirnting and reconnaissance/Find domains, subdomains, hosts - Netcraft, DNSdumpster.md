
## Online Tools
https://sitereport.netcraft.com/ - site technology, hosting history, risk rating  
https://dnsdumpster.com/ - DNS recon, subdomains, MX/TXT records, map  
https://www.whois.com/ - domain registration info  
https://who.is/ - alternative WHOIS  
https://centralops.net - WHOIS, DNS, traceroute all-in-one  
https://www.shodan.io - find exposed devices, services, banners  
https://censys.io - internet-wide scan data  
https://crt.sh - certificate transparency logs (find subdomains)  

## WHOIS Lookup (CLI)
`whois targetdomain.com` - registrant info, nameservers, creation/expiry dates  

## theHarvester - email, subdomain, IP gathering
`theHarvester -d targetdomain.com -l 200 -b google` - harvest from google  
`theHarvester -d targetdomain.com -l 200 -b linkedin` - harvest linkedin  
`theHarvester -d targetdomain.com -l 200 -b all` - all sources  
Sources: google, bing, linkedin, baidu, yahoo, netcraft, dnsdumpster, crtsh  

## Subfinder
`subfinder -d targetdomain.com` - fast passive subdomain enumeration  
`subfinder -d targetdomain.com -o subs.txt` - save to file  

## Sherlock - username OSINT
sherlock username  

## Maltego
- GUI OSINT tool, drag entity (domain/email/person) and run transforms  
- Maps relationships between infrastructure, people, domains  
- Exam: know it's used for **visualizing** footprinting data  

## Footprinting through Social Engineering
- Eavesdropping, shoulder surfing, dumpster diving  
- Social networking sites (LinkedIn, Facebook, Twitter)  

## Footprinting Countermeasures
- Restrict zone transfers  
- Disable directory listings  
- Use privacy services for WHOIS  
- Sanitize job postings (don't reveal tech stack)  

