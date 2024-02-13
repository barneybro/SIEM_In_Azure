<h1>Setting up a SIEM in Microsoft Azure</h1>


<h2>Description</h2>
In this lab, I setup Azure Sentinel (SIEM) and connect it to a live virtual machine acting as a honey pot. We will observe live attacks (RDP Brute Force) from all around the world. We will use a custom PowerShell script to look up the attackers Geolocation information and plot it on the Azure Sentinel Map!
<br />

<h2>Environments Used / Exposure to </h2>

- <b>Azure Portal</b> 
- <b>Azure Sentinal</b>
- <b>Kusto Query Language (KQL)</b>
- <b>Network Security Groups</b>

<h2>Program walk-through:</h2>

<p align="center">
Outline of SIEM: <br/>
<img src="https://i.imgur.com/WsjyTgN.png" height="80%" width="80%"/>
<br />
<br />
Create a Virtual Machine in Microsoft Azure / Create Log Analytics Workspace:  <br/>
<p align=center>While creating the VM, the user needs to add an inbound security rule that allows eveyrthing
into the it. Next, create a Log Analytics Workspace, go to Security Center to turn on Azure Defender, and turn off SQL servers. Go to the data collection tab next and select All Events.
Returning to the Log Analytics Workspace, you can now connect the log and the VM by selecting Virtual Machines on the left, selecting your VM and pressing connect. While
they are connecting go to Microsoft Sentinel, create a new MS, select the VM and add it. </p>
<img src="https://i.imgur.com/z94xnJI.png" height="80%" width="80%"/>
<br />
<br />
<p align="center">
Log Into VM through Remote Desktop Connection:<br/>
<p align="center">Copy the IP Address from the honeypot on the overview page and paste it into the RDC. Login with your username and password used to 
 create the VM, make sure to fail one of the events so you can see it in Event Viewer. Go to event viewer, Windows Logins, Security. This will show the attempts to login to the VM. In your VM
search for wf.msc which is Windows firewall. Click the properties and turn Domain, Public, and Private firewall states off. </p>
<img src="https://i.imgur.com/23Ys5FE.png" height="80%" width="80%"/>
<br />
<br />
<p align="center">
Windows Powershell ISE on VM: [Link](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) <br/>
 <p align="center">Open Windows Powershell On your VM. Create a new file and paste the source code from the link above inside, make sure to save to desktop.This code pretty much looks through Event Viewer,
 of people who failed to login, gets their geodata for them and creates a new log file for them. The only part you will have to change is the API Key at the top. You can go to ip.geolocation.io, sign up and you will get a free API key.
 The location of the log is in C:\ProgramData\, and it is called failed_rdp.log. </p>
<img src="https://i.imgur.com/iyPWvVn.png" height="80%" width="80%"/>
<br />
<br />
<p align="center">
Create a Custom Log in Log Analystics Workspace:  <br/>
<p align="center">Go to Log Analytics Workspace, there you will find the Tables tab. Click it then select Create a new custom log MMA based. It will then ask you for a file which you will need to get from the VM. Go into the VM and open the failed_rdp.log
log which will be in C:ProgramData\faided_rdp.log\. On your own desktop paste the contents into a new notepad file, select that file for the custom log. For the next part select Windows and for the path put C:ProgramData\faided_rdp.log\, or copy it from the VM.
Name it whatever you like and create. It will take time for the Log Analytics Workspace and VM to connect to be patient. To see it go to the Logs section and type in the name you used to create the custom log. Eventually it will show all the log info, but be patient. </p>
<img src="https://i.imgur.com/8iN0bBn.png" height="80%" width="80%"/>
<br />
<br />
 <p align="center">
Return to Microsoft Sentinel and set up map:  <br/>
  
  <p>Go to Microsoft Sentinel and click on your VM. To set up the GeoMap go to the Workbooks tab and add a new workbook. Click the edit and remove the two default widgets. Now add a new query, to save you time this is the query you will need to paste below -->
  FAILED_RDP_WITH_GEO_CL | extend username = extract(@"username:([^,]+)", 1, RawData), timestamp = extract(@"timestamp:([^,]+)", 1, RawData), latitude = extract(@"latitude:([^,]+)", 1, RawData), longitude = extract(@"longitude:([^,]+)", 1, RawData),
  sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData), state = extract(@"state:([^,]+)", 1, RawData), label = extract(@"label:([^,]+)", 1, RawData), destination = extract(@"destinationhost:([^,]+)", 1, RawData),country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost" | where sourcehost != "" | summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country.
<p align="center">All of the map settings should be in place, but feel free to change they how you'd like. Save the map and name it whatever you like and press done editing. You have your map now, refresh it to show new attacks!</p>
</p>
<img src="https://i.imgur.com/wbmhxv3.png" height="30%" width="50%"/>
<br />
<br />
<p align="center">
Final Map Results:  <br/>
<img src="https://i.imgur.com/Y3afS4G.png" height="80%" width="80%"/>
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
