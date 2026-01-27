
Pre-requisites:
--------------------
Local machine - Rocky linux 10 with Parrot OS VM

Server - Ubuntu server with libvirt (Ryzen 7 7700, 32GB, 3080, 1TB)

Images for VM's:
----------------------
- https://www.parrotsec.org/download/
- https://www.microsoft.com/lv-lv/software-download/windows11
- https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019
- https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022
- https://ubuntu.com/download/server/thank-you?version=24.04.3&architecture=amd64&lts=true

All VM's in - 10.10.10.0/24

*Add two interfaces for setup, one isolated and one for downloading necessary prerequisites*

| VM Name                | Role              | IP Address | VNC Port          |
| ---------------------- | ----------------- | ---------- | ----------------- |
| `ceh-parrot-attacker`  | Attacker          | 10.10.1.13 | 192.168.1.10:5910 |
| `ceh-win2022-dc`       | Domain Controller | 10.10.1.22 | 192.168.1.10:5911 |
| `ceh-win2019-target`   | Target Server     | 10.10.1.19 | 192.168.1.10:5912 |
| `ceh-win11-standalone` | Standalone Client | 10.10.1.11 | 192.168.1.10:5913 |
| `ceh-win11-domain`     | Domain Client     | 10.10.1.40 | 192.168.1.10:5914 |
| `ceh-ubuntu-target`    | Linux Target      | 10.10.1.9  | 192.168.1.10:5915 |
| `ceh-android`          | Android           | 10.10.1.50 | 192.168.1.10:5916 |

## Default Credentials

| OS        | Username      | Password  |
| --------- | ------------- | --------- |
| Parrot OS | parrot        | parrot    |
| Windows   | Administrator | yes       |
| Ubuntu    | ubuntu        | yesubuntu |
| Windows   | Admin         | same      |




| **VM Name**          | **OS**        | **Role**          | **Setup (Roles / Users / Software)**                 |
| -------------------- | ------------- | ----------------- | ---------------------------------------------------- |
| ceh-parrot-attacker  | Parrot OS     | Attacker          | Attacking tools, browser, nmap, network access       |
| ceh-win2022-dc       | Windows 2022  | Domain Controller | AD DS, DNS, domain created, Administrator account    |
| ceh-win2019-target   | Windows 2019  | Server Target     | IIS installed, HTTP enabled, firewall disabled       |
| ceh-win11-standalone | Windows 11    | Standalone Client | Base OS only                                         |
| ceh-win11-domain     | Windows 11    | Domain Client     | Joined domain, DNS pointing to DC                    |
| ceh-ubuntu-target    | Ubuntu Server | Linux Target      | Apache, PHP, MariaDB, DVWA installed and initialized |
| ceh-android          | Android       | Mobile Target     | Imported VM only                                     |


