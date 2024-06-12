# Azure-Sentinel- 

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <h1>SIEM Lab</h1>
    <p> Summary:
SIEM stands for Security Information and Event Management System. It is a solution that helps organizations detect, analyze, and respond to security threats before they harm business operations. It is a tool that collects event log data from a range of sources within a network such as Firewalls, IDS/IPS, Identity solutions, etc. This allows the security professionals to monitor, prioritize and remediate potential threats in real-time. A honeypot is a security mechanism that creates a virtual trap in a controlled and safe environment to lure attackers. An intentionally compromised computer system to study how attackers work, examine different types of threats and improve security policies. This lab's purpose is to understand how to collect honeypot attack log data, query it in a SIEM and display it in a manner that is easy to understand such data. In this case it will be displayed in a world map by event count and geolocation.

Learning Objectives:
Configuration & Deployment of Azure resources such as virtual machines, Log Analytics Workspaces, and Azure Sentinel
Hands-on experience and working knowledge of a SIEM Log Management Tool (Microsoft's Azure Sentinel)
Understand Windows Security Event logs
Utilization of KQL to query logs
Display attack data on a dashboard with Workbooks (World Map)
Tools & Requirements:
Microsoft Azure Subscription
Azure Sentinel
Kusto Query Language (KQL - Used to build world map)
Network Security Groups (Layer 4/3 Firewall in Azure)
Remote Desktop Protocol (RDP)
3rd Party API: :.io:</p>
    <img src="https://i.imgur.com/uH7VDcm.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">
    Overview:
     <img src="https://i.imgur.com/mwyNJTr.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 1: Create a Microsoft Azure Subscription: Azure
<img src="https://i.imgur.com/JXF2FPt.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 2: Create a Honeypot Virtual Machine


<img src="https://i.imgur.com/HxXD4wK.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Basics
Go to portal.azure.com
Search for "virtual machines"
Create > Azure virtual machine
Project details
Create new resource group and name it (honeypotlab)
A resource group is a collection of resources that share the same lifecycle, permissions, and policies.

Instance details
Name your VM (honeypot-vm)
Select a recommended region ((US) East US 2)
Availability options: No infrastructure redundancy required
Security type: Standard
Image: Windows 10 Pro, version 21H2 - x62 Gen2
VM Architecture: x64
Size: Default is okay (Standard_D2s_v3 – 2vcpus, 8 GiB memory)
Administrator account
Create a username and password for virtual machine
IMPORTANT NOTE: These credentials will be used to log into the virtual machine (Keep them handy)

Inbound port rules
Public inbound ports: Allow RDP (3389)
Licensing
Confirm licensing
Select Next : Disks >

<img src="https://i.imgur.com/DBy9n5m.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Disks
Leave all defaults
Select Next : Networking >
Networking
Network interface
NIC network security group: Advanced > Create new
A network security group contains security rules that allow or deny inbound network traffic to, or outbound network traffic from, the virtual machine. In other words, security rules management.

Remove Inbound rules (1000: default-allow-rdp) by clicking three dots
Add an inbound rule
Destination port ranges: * (wildcard for anything)
Protocol: Any
Action: Allow
Priority: 100 (low)
Name: Anything (ALLOW_ALL_INBOUND)
Select Review + create

Step 3: Create a Log Analytics Workspace
Search for "Log analytics workspaces"
Select Create Log Analytics workspace
Put it in the same resource group as VM (honeypotlab)
Give it a desired name (honeypot-log)
Add to same region (East US 2)
Select Review + create

<img src="https://i.imgur.com/JH12U0F.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 4: Configure Microsoft Defender for Cloud
Search for "Microsoft Defender for Cloud"
Scroll down to "Environment settings" > subscription name > log analytics workspace name (log-honeypot)

<img src="https://i.imgur.com/YneLnZ7.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Settings | Defender plans
Cloud Security Posture Management: ON
Servers: ON
SQL servers on machines: OFF
Hit Save

<img src="https://i.imgur.com/4aNHJj7.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Settings | Data collection
Select "All Events"
Hit Save

Step 5: Connect Log Analytics Workspace to Virtual Machine
Search for "Log Analytics workspaces"
Select workspace name (log-honeypot) > "Virtual machines" > virtual machine name (honeypot-vm)
Click Connect

<img src="https://i.imgur.com/0kJ8Msg.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 6: Configure Microsoft Sentinel
Search for "Microsoft Sentinel"
Click Create Microsoft Sentinel
Select Log Analytics workspace name (honeypot-log)
Click Add

<img src="https://i.imgur.com/KYEzB8T.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 7: Disable the Firewall in Virtual Machine
Go to Virtual Machines and find the honeypot VM (honeypot-vm)
By clicking on the VM copy the IP address
Log into the VM via Remote Desktop Protocol (RDP) with credentials from step 2
Accept Certificate warning
Select NO for all Choose privacy settings for your device
Click Start and search for "wf.msc" (Windows Defender Firewall)
Click "Windows Defender Firewall Properties"
Turn Firewall State OFF for Domain Profile Private Profile and Public Profile
Hit Apply and Ok
Ping VM via Host's command line to make sure it is reachable ping -t <VM IP>

<img src="https://i.imgur.com/DhESpQ5.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 8: Scripting the Security Log Exporter
In VM open Powershell ISE
Set up Edge without signing in
Copy Powershell script into VM's Powershell 
Select New Script in Powershell ISE and paste script
Save to Desktop and give it a name (Log_Exporter)


<img src="https://i.imgur.com/K6vzWAz.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Make an account with Free IP Geolocation API and Accurate IP Lookup Database
This account is free for 1000 API calls per day. Paying 15.00$ will allow 150,000 API calls per month.

Copy API key once logged in and paste into script line 2: $API_KEY = "<API key>"
Hit Save
Run the PowerShell ISE script (Green play button) in the virtual machine to continuously produce log data

<img src="https://i.imgur.com/LSQTeCy.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

The script will export data from the Windows Event Viewer to then import into the IP Geolocation service. It will then extract the latitude and longitude and then create a new log called failed_rdp.log in the following location: C:\ProgramData\failed_rdp.log

Step 9: Create Custom Log in Log Analytics Workspace
Create a custom log to import the additional data from the IP Geolocation service into Azure Sentinel
Search "Run" in VM and type "C:\ProgramData"
Open file named "failed_rdp" hit CTRL + A to select all and CTRL + C to copy selection
Open notepad on Host PC and paste contents
Save to desktop as "failed_rdp.log"
In Azure go to Log Analytics Workspaces > Log Analytics workspace name (honeypot-log) > Custom logs > Add custom log
Sample
Select Sample log saved to Desktop (failed_rdp.log) and hit Next
Record delimiter
Review sample logs in Record delimiter and hit Next
Collection paths
Type > Windows
Path > "C:\ProgramData\failed_rdp.log"
Details
Give the custom log a name and provide description (FAILED_RDP_WITH_GEO) and hit Next
Hit Create

<img src="https://i.imgur.com/7ibnkFk.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 10: Query the Custom Log
In Log Analytics Workspaces go to the created workspace (honeypot-log) > Logs
Run a query to see the available data (FAILED_RDP_WITH_GEO_CL)
May take some time for Azure to sync VM and Log Analytics

<img src="https://i.imgur.com/EeBNLhx.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 11: Extract Fields from Custom Log
The RawData within a log contains information such as latitude, longitude, destinationhost, etc. Data will have to be extracted to create separate fields for the different types of data

Right click any of the log results
Select Extract fields from 'FAILED_RDP_WITH_GEO_CL'
Highlight ONLY the value after the ":"
Name the Field Title the name of the field of the value
Under Field Type select the appropriate data type
Hit Extract
If the search results data looks good click the Save extraction button
Do this for ALL available fields in RawData
NOTE: If one of the search results is not correct select Modify this highlight (upper right corner of result) and highlight the correct value. Otherwise go to Custom logs > Custom fields Accept warning of unsaved edits and delete field. Redo extraction for deleted field.

<img src="https://i.imgur.com/svMBZn9.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Step 12: Map Data in Microsoft Sentinel
Go to Microsoft Sentinel to see the Overview page and available events
Click on Workbooks and Add workbook then click Edit
Remove default widgets (Three dots > Remove)
Click Add > Add query
Copy/Paste the following query into the query window and Run Query

FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""

Kusto Query Language (KQL) - Azure Monitor Logs is based on Azure Data Explorer. The language is designed to be easy to read and use with some practice writing queries and basic guidance.

Once results come up click the Visualization dropdown menu and select Map
Select Map Settings for additional configuration
Layout Settings
Location info using > Latitude/Longitude
Latitude > latitude_CF
Longitude > longitude_CF
Size by > event_count
Color Settings
Coloring Type: Heatmap
Color by > event_count
Aggregation for color > Sum of values
Color palette > Green to Red
Metric Settings
Metric Label > label_CF
Metric Value > event_count
Select Apply button and Save and Close
Save as "Failed RDP World Map" in the same region and under the resource group (honeypotlab)
Continue to refresh map to display additional incoming failed RDP attacks
NOTE: The map will only display Event Viewer's failed RDP attempts and not all the other attacks the VM may be receiving.

<img src="https://i.imgur.com/jeBKbnL.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

Event Viewer Displaying Failed RDP logon attemps. EventID 4625

 <img src="https://i.imgur.com/mAjySoX.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

 Custom Powershell script parsing data from 3rd party API

 <img src="https://i.imgur.com/gEaNsKH.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">

 Step 13: Deprovision Resources
VERY IMPORTANT - Do NOT skip!

Search for "Resource groups" > name of resource group (honeypotlab) > Delete resource group
Type the name of the resource group ("honeypotlab") to confirm deletion
Check the Apply force delete for selected Virtual machines and Virtual machine scale sets box
Select Delete

  <img src="https://i.imgur.com/6ABE9Cy.png" alt="Lab Screenshot" style="max-width: 100%; height: auto;">
  







    
