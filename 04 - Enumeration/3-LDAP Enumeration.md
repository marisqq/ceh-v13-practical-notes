
Active directory explorer  
-----------------------------------

![](../img/Pasted%20image%2020260129213350.png)  

Modify strings:  
 ![](../img/Pasted%20image%2020260129213629.png)  

------------------------------------------------------

Other software:  
**Softerra LDAP Administrator** (https://www.ldapadministrator.com), **LDAP Admin Tool** (https://www.ldapsoft.com),  
**LDAP Account Manager** (https://www.ldap-account-manager.org), and **LDAP Search** (https://securityxploded.com) to perform LDAP enumeration on the target.  

------------------------------------------

## LDAP CLI Enumeration (Linux)

`ldapsearch -x -h [ip] -b "dc=target,dc=com"` - anonymous bind, dump directory  
`ldapsearch -x -h [ip] -b "dc=target,dc=com" "objectclass=user"` - enumerate users  
`ldapsearch -x -h [ip] -b "dc=target,dc=com" "objectclass=computer"` - enumerate computers  

-x = simple authentication  
-h = host  
-b = base DN (search base)  

## Nmap LDAP Scripts
`nmap -p 389 --script ldap-search [ip]` - enumerate LDAP  
`nmap -p 389 --script ldap-brute [ip]` - brute force LDAP  

## Key LDAP Ports
| Port | Protocol |
|------|----------|
| 389 | LDAP (cleartext) |
| 636 | LDAPS (SSL/TLS) |
| 3268 | Global Catalog |
| 3269 | Global Catalog SSL |
