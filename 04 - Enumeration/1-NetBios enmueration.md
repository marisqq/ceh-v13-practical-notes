
| Name      | NetBIOS Code | Type   | Information obtained                         |
| --------- | ------------ | ------ | -------------------------------------------- |
| host name | <00>         | UNIQUE | Hostname                                     |
| domain    | <00>         | Group  | Domain name                                  |
| host name | <03>         | UNIQUE | Messenger service                            |
| username  | <03>         | UNIQUE | Messenger service running for logged in user |
| host name | <20>         | UNIQUE | Server service running                       |
| domain    | 1B           | UNIQUE | Domain Master browser name                   |
| domain    | 1E           | Group  | Browser service elections                    |


-----------------------------------------------



`nbtstat` -a 10.10.10.1  | -a displays NetBios name table  

![](../img/Pasted%20image%2020260129201308.png)  


`nbtstat` -c | display netbios name catche  

----------------------------------------------------------------

`net use` | displays information about target such as connection status and shared drives  

![](../img/Pasted%20image%2020260129203646.png)  


---------------------------------------------------------------

## Enum4linux (Linux - SMB/NetBIOS enumeration)

`enum4linux -a [ip]` | full enumeration (users, shares, groups, OS, policies)  
`enum4linux -u "" -p "" [ip]` | null session enumeration  
`enum4linux -U [ip]` | enumerate users  
`enum4linux -S [ip]` | enumerate shares  
`enum4linux -G [ip]` | enumerate groups  
`enum4linux -P [ip]` | password policy  

---------------------------------------------------------------

## SMB Null Session (Windows)

Establish null session to target:  
`net use \\10.10.1.22\IPC$ "" /u:""`  

Then enumerate:  
`net view \\10.10.1.22` - shared resources  

---------------------------------------------------------------

## Nmap NetBIOS/SMB Scripts

`nmap -sV -p 139,445 --script nbstat.nse [ip]` - NetBIOS info  
`nmap --script smb-enum-shares [ip]` - enumerate shares  
`nmap --script smb-enum-users [ip]` - enumerate users  
`nmap --script smb-os-discovery [ip]` - OS info via SMB  

---------------------------------------------------------------

## Global Network Inventory / Hyena
- Windows GUI tools for NetBIOS enumeration  
- Can browse network, view shares, users, groups  
