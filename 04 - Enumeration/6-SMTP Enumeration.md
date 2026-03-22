## SMTP Enumeration (Port 25)  
  
SMTP commands to verify users on mail server:  
  
### Manual via Telnet/Netcat  
```  
telnet [ip] 25  
VRFY admin@target.com      → verifies if user exists (252=exists, 550=not found)  
EXPN admin@target.com      → expands mailing list members  
RCPT TO:admin@target.com   → verifies recipient (after MAIL FROM)  
```  
  
### Nmap SMTP Scripts  
`nmap -p 25 --script smtp-enum-users [ip]` - enumerate valid users  
`nmap -p 25 --script smtp-open-relay [ip]` - check open relay  
`nmap -p 25 --script smtp-commands [ip]` - list supported commands  
  
### smtp-user-enum (Kali)  
`smtp-user-enum -M VRFY -U /usr/share/wordlists/users.txt -t [ip]`  
`smtp-user-enum -M RCPT -U users.txt -t [ip]`  
  
-M = method (VRFY, EXPN, RCPT)  
-U = username wordlist  
-t = target  
