## FTP Enumeration (Port 21)  
  
### Banner Grabbing  
`ftp [ip]` - connect and check banner  
`nmap -sV -p 21 [ip]` - version detection  
  
### Anonymous Login  
```  
ftp [ip]  
Username: anonymous  
Password: (blank or any email)  
```  
  
### Nmap FTP Scripts  
`nmap -p 21 --script ftp-anon [ip]` - check anonymous login  
`nmap -p 21 --script ftp-brute [ip]` - brute force credentials  
`nmap -p 21 --script ftp-bounce [ip]` - check FTP bounce attack  
  
### Hydra FTP Brute Force  
`hydra -L users.txt -P passwords.txt ftp://[ip]`  
