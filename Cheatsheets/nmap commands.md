
| can Type / Purpose                        | Command Example                                                                                      |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| TARGET SELECTION                          |                                                                                                      |
| Scan a single IP                          | nmap 192.168.1.1                                                                                     |
| Scan a host                               | nmap www.testhostname.com                                                                            |
| Scan a range of IPs                       | nmap 192.168.1.1-20                                                                                  |
| Scan a subnet                             | nmap 192.168.1.0/24                                                                                  |
| Scan targets from a text file             | nmap -iL list-of-ips.txt                                                                             |
| Scan IPv6                                 | nmap -6 fe80::1                                                                                      |
| PORT SELECTION                            |                                                                                                      |
| Scan a single Port                        | nmap -p 22 192.168.1.1                                                                               |
| Scan a range of ports                     | nmap -p 1-100 192.168.1.1                                                                            |
| Scan 100 most common ports (Fast)         | nmap -F 192.168.1.1                                                                                  |
| Scan all 65535 ports                      | nmap -p- 192.168.1.1                                                                                 |
| Scan specific ports                       | nmap -p 22,80,443 192.168.1.1                                                                        |
| Scan ports by name                        | nmap -p http,https 192.168.1.1                                                                       |
| PORT SCAN TYPES                           |                                                                                                      |
| TCP connect scan                          | nmap -sT 192.168.1.1                                                                                 |
| TCP SYN scan (default, requires root)     | nmap -sS 192.168.1.1                                                                                 |
| UDP port scan                             | nmap -sU -p 123,161,162 192.168.1.1                                                                  |
| Fast scan. Skip discovery (-Pn, -F)       | nmap -Pn -F 192.168.1.1                                                                              |
| TCP ACK scan (Firewall rule mapping)      | nmap -sA 192.168.1.1                                                                                 |
| TIMING & PERFORMANCE CONTROLS             |                                                                                                      |
| Increase scan speed (minimum packet rate) | nmap --min-rate 5000 192.168.1.1                                                                     |
| Throttle scan speed (maximum packet rate) | nmap --max-rate 1000 192.168.1.1                                                                     |
| Limit retries (avoid slow networks)       | nmap --max-retries 2 192.168.1.1                                                                     |
| Set a timeout per host                    | nmap --host-timeout 30s 192.168.1.1                                                                  |
| HOST DISCOVERY                            |                                                                                                      |
| No scan - list targets only*              | nmap -sL 192.168.1.0/24                                                                              |
| Disable host discovery - port scan only   | nmap -Pn 192.168.1.1                                                                                 |
| Disable port scan (Ping scan)             | nmap -sn 192.168.1.0/24                                                                              |
| TCP SYN discovery on port 443             | nmap -PS443 192.168.1.1                                                                              |
| TCP ACK discovery on port 80              | nmap -PA80 192.168.1.1                                                                               |
| UDP discovery on port 53                  | nmap -PU53 192.168.1.1                                                                               |
| SERVICE AND OS DETECTION                  |                                                                                                      |
| Standard Version detection                | nmap -sV 192.168.1.1                                                                                 |
| Aggressive Service Detection              | nmap -sV --version-intensity 5 192.168.1.1                                                           |
| Lighter banner grabbing detection         | nmap -sV --version-intensity 0 192.168.1.1                                                           |
| Detect OS and Services                    | nmap -A 192.168.1.1                                                                                  |
| OS detection only                         | nmap -O 192.168.1.1                                                                                  |
| Aggressive scan and traceroute            | nmap -A -T4 192.168.1.1                                                                              |
| OUTPUT FORMATS                            |                                                                                                      |
| Save default output to file               | nmap -oN outputfile.txt 192.168.1.1                                                                  |
| Save results as XML                       | nmap -oX outputfile.xml 192.168.1.1                                                                  |
| Save results in a format for grep         | nmap -oG outputfile.txt 192.168.1.1                                                                  |
| Save in all formats                       | nmap -oA outputfile 192.168.1.1                                                                      |
| NSE SCRIPTING                             |                                                                                                      |
| Scan using default safe scripts           | nmap -sV -sC 192.168.1.1                                                                             |
| Quick web recon                           | nmap -p 80,443 --script=http-title,http-server-header 192.168.1.1                                    |
| HTTP Security headers (HSTS, CSP, etc)    | nmap -p 80,443 --script=http-security-headers 192.168.1.1                                            |
| Get help for a script                     | nmap --script-help=http-enum                                                                         |
| Scan using a specific NSE script          | nmap -sV -p 22 --script=ssh-hostkey 192.168.1.1                                                      |
| Scan with a set of scripts                | nmap -sV --script=smb* 192.168.1.1                                                                   |
| Scan for UDP DDoS reflectors              | nmap -sU -A -Pn -n -pU:19,53,123,161 --script=ntp-monlist,dns-recursion,snmp-sysdescr 192.168.1.0/24 |
| Gather page titles from HTTP services     | nmap --script=http-title 192.168.1.0/24                                                              |
| Get HTTP headers of web services          | nmap --script=http-headers 192.168.1.0/24                                                            |
| Find web apps from known paths            | nmap --script=http-enum 192.168.1.0/24                                                               |
| SSH Host Key Collection                   | nmap -p 22 --script=ssh-hostkey,ssh-auth-methods 192.168.1.1                                         |
| Find Information about IP address         | nmap --script=asn-query,whois,ip-geolocation-maxmind 192.168.1.0/24                                  |
| Check for SSL Heartbleed (legacy example) | nmap -sV -p 443 --script=ssl-heartbleed 192.168.1.0/24                                               |
| TLS cert + cipher information             | nmap -p 443 --script=ssl-cert,ssl-enum-ciphers 192.168.1.1                                           |