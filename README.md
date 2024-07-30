# Active Directory Project

In this project, I'll be setting up an Active Directory (home lab) that includes Splunk, Kali Linux & Atomic Red Team, Windows 10 and Windows Server 2022. I'll explore how a domain environment works, learn how to ingest events to a SIEM and generate telemetry related to attacks seen in the wild to help you detect them in the future.

Special Thanks to Steven over at https://www.youtube.com/@MyDFIR for providing an amazing Tutorial on how to get Started with this lab.

## Logical Diagram

The first step was to come up with a logical diagram in order to understand how the data would flow between the nodes. I used draw.io for this since it was the easiest option to come up with a logical diagram

![Logical Diagram](https://github.com/user-attachments/assets/7ea03bf1-1113-43bf-9f52-1756cb94d174)

Once I was done creating the logical diagrams, it was time to move on and set up the Devices. I decided to use VMWare to accomplish this since I was most comfortable with that Virtualization Software.

## Setting up Virtual Machines

### Splunk Server (Ubuntu 22.04.4 Server)

I decided to use Ubuntu server to run the Splunk Indexer to receive logs from the Splunk Forwarder installed in the Domain Computers (Windows 10 and Windows Server 2022). The setup for the VM was pretty simple, I went with the default settings except for a slight change in the network settings where I set up a static IP for the Server as can be seen below.

![Splunk Server Network Settings](https://github.com/user-attachments/assets/fe4c2972-567a-4bf2-9f90-00cff47aa132)

All that is left to do with this is to install the Splunk Indexer.

### Windows Server 2022

I decided to use Windows Server 2022 to act as the Domain Controller for our Virtual Environment which will have installed Splunk Forwarder and Sysmon to send generate and send logs over to our Splunk Indexer.

I installed the Windows Server 2022 Standard (Desktop Experience) so as to get a GUI to interact with so as to make things easier.

![Microsoft Server 2022 Install Settings](https://github.com/user-attachments/assets/158a76f6-62c2-472b-b3ab-be1958a609c8)

The Virtual Hardware was pretty standard since this was just a Home Lab

![Microsoft Server 2022 VM Settings](https://github.com/user-attachments/assets/75b259a7-4b9a-4190-9e66-382a88a8a7d7)

### Windows 10 User

I decided to use a Windows 10 host, since I had that VM already set up with basic configurations from my previous Course related Assignments. All that was needed was to install the Splunk Universal Forwarder.

![Windows 10 User](https://github.com/user-attachments/assets/b46a4b49-867f-4fd4-b217-565df899b868)

### Kali Linux (Attacker)

For the attacker I decided to go with Kali Linux which is the most popular OS for Attacking. I went with the basic setup for this and downloaded the VMWare image from Kali's website.

## Installing Software

### Splunk Indexer (Ubuntu 22.04 Server)

The Splunk Server was going to be hosting the Splunk Indexer so I installed the Splunk Enterprise 9.3.0 onto the Server.

![Splunk Indexer](https://github.com/user-attachments/assets/33eb99bb-da49-4b82-86e5-9b25f432c62b)


### Splunk Forwarder (Windows 10 and Windows Server 2022)

The Splunk Forwarder was to be installed on the Windows 10 User and the Windows Server 2022. I went with the most default settings

![Splunk Forwarder](https://github.com/user-attachments/assets/f57fcc5a-331d-46b6-a447-9efc355cf4fd)

Setting the indexer to the Splunk Server (192.168.10.10)

![Splunk Indexer Settings](https://github.com/user-attachments/assets/5767a394-42a5-4b5a-9692-7fe87ec7fc5e)

After installing the forwarder what was left was to give privilege to the SplunkForwarder service to actually send the logs to the indexer. This was done by changing the user that was running the service from NT SERVICE\SplunkForwarder to Local System Account

![NT SERVICE](https://github.com/user-attachments/assets/33c7a0c0-40f3-4b36-aceb-e94bf5dd64ae)


![Local System Account](https://github.com/user-attachments/assets/073c3666-f12f-4318-9fdb-e44a34225bc9)


There was one last thing to configure. What logs to be sent to the indexer, this was done by adding a config file to "C:\Program Files\SplunkUniversalForwarder\etc\system\local" and name it inputs.conf

`
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
`

This config would send Application, Security, System, and Sysmon data to the endpoint index onto the Splunk Indexer

