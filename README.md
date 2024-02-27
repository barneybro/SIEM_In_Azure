<h1>Setting up a SIEM in Microsoft Azure</h1>


<h2>Description</h2>
In this lab, I setup Azure Sentinel (SIEM) and connect it to a live virtual machine acting as a honey pot. You will observe live attacks (RDP Brute Force) from all around the world. You will use a custom PowerShell script to look up the attackers Geolocation information and plot it on the Azure Sentinel Map!
<br />

<h2>Environments Used / Exposure to </h2>

- <b>Azure Portal</b> 
- <b>Azure Sentinal</b>
- <b>Kusto Query Language (KQL)</b>
- <b>Network Security Groups</b>

<h2>Program walk-through:</h2>

<p align="center">
<b>Outline of SIEM: </b> <br/>
<img src="https://i.imgur.com/WsjyTgN.png" height="80%" width="80%"/>
<br />
<br />
<b>Create a Virtual Machine in Microsoft Azure / Create Log Analytics Workspace:</b>  <br/>
<p align=center>
 
 - While creating the VM, add an inbound security rule allowing everything into it.
- Create a Log Analytics Workspace.
- In Security Center, turn on Azure Defender and turn off SQL servers.
- In the data collection tab, select All Events.
- Return to the Log Analytics Workspace, connect the log and the VM by selecting Virtual Machines on the left, selecting your VM, and pressing connect.
- While they are connecting, go to Microsoft Sentinel, create a new MS, select the VM, and add it.</p>
<img src="https://i.imgur.com/z94xnJI.png" height="80%" width="80%"/>
<br />
<br />
<p align="center">
<b>Log Into VM through Remote Desktop Connection:</b><br/>
<p align="center">

- Copy the IP Address from the honeypot on the overview page and paste it into the RDC.
- Login with your username and password used to create the VM; make sure to fail one of the events to see it in Event Viewer.
- Go to Event Viewer, navigate to Windows Logins, and check Security to view login attempts to the VM.
- In your VM, search for wf.msc (Windows firewall), click properties, and turn off Domain, Public, and Private firewall states. </p>
<img src="https://i.imgur.com/23Ys5FE.png" height="80%" width="80%"/>
<br />
<br />
<p align="center">
<b>Windows Powershell ISE on VM: Link --></b> https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1 <br/>
 <p align="center">
  
- Open Windows PowerShell on your VM.
- Create a new file and paste the source code from the link above, ensuring to save it to the desktop.
- The code looks through Event Viewer for failed logins, retrieves geodata, and creates a new log file.
- Change the API Key at the top by signing up at ip.geolocation.io to get a free key.
- The log is located in C:\ProgramData\ and is named failed_rdp.log. </p>
<img src="https://i.imgur.com/iyPWvVn.png" height="80%" width="80%"/>
<br />
<br />
<p align="center">
<b>Create a Custom Log in Log Analystics Workspace:</b>  <br/>
<p align="center">
 
- Go to Log Analytics Workspace and navigate to the Tables tab.
- Click on it and select "Create a new custom log MMA based."
- Provide the required file, obtained from the VM. Do this by opening the failed_rdp.log file in C:\ProgramData\failed_rdp.log on the VM, paste its contents into a new notepad file on your desktop, and select that file for the custom log.
- For the next step, choose Windows, and for the path, put C:\ProgramData\failed_rdp.log, or copy it from the VM.
- Name the log whatever you prefer and create it.
- It will take time for the Log Analytics Workspace and VM to connect, so be patient.
- To view it, go to the Logs section, type in the name you used for the custom log, and eventually, it will display all the log information. It may take 10-15 minutes. </p>
<img src="https://i.imgur.com/8iN0bBn.png" height="80%" width="80%"/>
<br />
<br />
 <p align="center">
<b>Return to Microsoft Sentinel and set up map:</b>  <br/>
  
  <p>
   
- In Microsoft Sentinel, click on your VM.
- To set up the GeoMap, go to the Workbooks tab and add a new workbook.
- Click edit and remove the two default widgets.
- Add a new query, and paste the following:
Copy code:

FAILED_RDP_WITH_GEO_CL | extend username = extract(@"username:([^,]+)", 1, RawData), timestamp = extract(@"timestamp:([^,]+)", 1, RawData), latitude = extract(@"latitude:([^,]+)", 1, RawData), longitude = extract(@"longitude:([^,]+)", 1, RawData),
sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData), state = extract(@"state:([^,]+)", 1, RawData), label = extract(@"label:([^,]+)", 1, RawData), destination = extract(@"destinationhost:([^,]+)", 1, RawData),country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost" | where sourcehost != "" | summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country
- Ensure that all map settings are in place, but feel free to customize them as you'd like.
- Save the map, name it as you prefer, and press done editing.
- You now have your map; refresh it to display new attacks.</p>
</p>
<img src="https://i.imgur.com/wbmhxv3.png" height="30%" width="50%"/>
<br />
<br />
<p align="center">
<b>Example Final Map Results:</b><br/>
 <p align="center">These results are from an hour of running, with many attacks from Vietnam and Japan. The ones in the U.S. are from my failed attempts. If you decided to run it longer you will get results from all over the world. Very cool!</p>
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
