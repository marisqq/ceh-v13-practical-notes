
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

