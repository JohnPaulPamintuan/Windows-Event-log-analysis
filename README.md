# Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server
ETW (Event tracing for Windows): https://github.com/JohnPaulPamintuan/ETW

## Description:
- <b>Windows Event Logs</b> :  are an intrinsic part of the Windows Operating System, storing logs from different components of the system including the system itself, applications running on it, ETW providers, services, and others. Windows event logging offers comprehensive logging capabilities for application errors, security events, and diagnostic information.

## Tool Used: 
- <b>Event logs </b>: Built-in logging mechanism in Windows for recording system events, including security events, application events, and system events.
- <b>Event Viewer</b> : A Microsoft Management Console (MMC) application that allows users to view and analyze event logs on Windows systems.
- <b>Sysmon (System Monitor) </b> :  System Monitor (Sysmon) is a Windows system service and device driver that remains resident across system reboots to monitor and log system activity to the Windows event log. Sysmon provides detailed information about process creation, network connections, changes to file creation time, and more.

## Purposes: 
- Enhance security visibility on Windows Servers.
- Detect malicious activities like DLL hijacking.

# Setting Up the Environment: 
## <b>Accessing Event Viewer</b>:
- Open Event Viewer

     <img src="https://i.imgur.com/mUTcNU7.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

- Familiarize yourself with the main log categories:
  -  <b>Application Logs </b>: Logs related to applications and services.
  -  <b>Security Logs </b>: Logs related to security events (e.g., logon attempts).
  -  <b>System Logs </b>: Logs related to Windows system events.

      <img src="https://i.imgur.com/cs76zjJ.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

### Key Event Logs to Monitor
- <b><ins>Security Log:</b></ins>
  -  <b>Event ID 4624 </b>: Successful logon.
  -  <b>Event ID 4625 </b>: Failed logon attempts (indicating potential brute force attacks).
  -  <b>Event ID 4648 </b>: A logon attempt using explicit credentials.
- <b><ins>System Log:</b></ins>
  -  <b>Event ID 6005 </b>: The Event Log service was started (system boot).
  -  <b>Event ID 6006 </b>: The Event Log service was stopped (system shutdown).
- <b><ins>Application Log:</b></ins>
  - Monitor for application  <b>errors or warnings </b> that may indicate malicious activity.

# <b>Detecting DLL hijacking with Sysmon logs </b>:
## <b>Installing Sysmon </b>:
- Download Sysmon from Microsoft Sysinternals. Here are the links below: 
  - [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
  - [Sysmon Configuration](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml)

- Run the following command to install using <ins>Windows Powershell as Administrator</ins>
- We want to make sure that we are in the same direcrtory as Sysmon. In this case: "C:\User\Johnp\documents\donwloads\Sysmon"

     <img src="https://i.imgur.com/KMA9e6M.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
     <img src="https://i.imgur.com/OXddcZc.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

### Configuration    
- Modify <ins>sysmon-config.xml</ins> to include rules for detecting suspicious activities.
- Enable logging for DLL loading and loading events specifically.
  -  Run this following command or go from the sysmonconfig-export.xml file, you need to open with notepad and change the ‘include’ to ‘exclude’

    <img src="https://i.imgur.com/YztyYRa.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    <img src="https://i.imgur.com/KvvJRgJ.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    
# DLL hijacking & detection in action
- In this walkthrough, I’ll hijack a DLL used by calc.exe. Let’s first start of by identifying the DLLs loaded by calc.exe, I will be doing this by simply running Process Monitor and running calc.exe. 

     <img src="https://i.imgur.com/lf20S9C.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
Highlighted above is C:\Windows\System32\wininet.dll , which is loaded into the memory of the calc.exe process.

- Now we have identified a possible DLL that we can recreate with our own rouge one let’s get on with the hijacking. I cheated and used this prebuilt PoC DLL from Stephen Fewer, this will not do anything malicious, but instead just display a ‘Hello from DLLMain!’ pop up box if it has been successfully hijacked. I then used the ‘Relative Path DLL’ method mentioned earlier and just copied calc.exe and the rouge DLL to my own folder. I also renamed my rouge DLL to wininet.dll, this way it would masquerade as a legitimate one and load successfully.
- This time when I double-clicked calc.exe to execute, instead of the calculator opening, I got prompted with this ‘Hello from DLLMain!’ message. We can see on process monitor this time, wininet.dll (the rouge one) was loaded — but not from the file location C:\Windows\System32\wininet.dll !
  
   <img src="https://i.imgur.com/EHHHfqY.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>


### Collecting Event Logs
- Open Event Viewer and navigate to <b>Applications and Services Logs > Microsoft > Windows > Sysmon > Operational</b>

    <img src="https://i.imgur.com/RtH4E85.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

### Identifying Important Events:
- Look for:
  - <b>Event ID 1 </b>: Process creation.
  - <b>Event ID 7 </b>: Image loaded (for DLL loading).
  - <b>Event ID 10 </b>: Process accessed.

    <img src="https://i.imgur.com/0Sh2MU9.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

- Filtering Event:  
    <img src="https://i.imgur.com/nfbEOLu.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    <img src="https://i.imgur.com/PjOcEv2.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

- In Event Viewer we can observe there is a lot events. As we are looking for evidence of DLL hijacking I will be searching for Event ID 7, ImageLoad. This event logs all instances of DLLs being loaded into a programs memory.

- Here we can see the forensic artefact of DLL hijacking residing in our sysmon log. Let’s quickly compare this to the the log entry of the the legitimate DLL loaded by a legitimate calc.exe
  
    <img src="https://i.imgur.com/pI1URyz.png" height="80%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    <img src="https://i.imgur.com/zVLPLVC.png" height="80%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
 - Comparing the two can help us identify useful IOCs of DLL hijacking:
   - Our rouge DLL is not signed, however, the legit DLL is signed by ‘Microsoft Windows’. This is a big indicator that something dodgy may be going on
   - The rouge DLL (and actual calc.exe!) is loaded from C:\Users\cyben\...\ReflectiveDLLInjection , whereas legitimate DLLs will almost always be found in C:\Windows\System32 or ProgramFiles directories. This is an indicator of Relative Path DLL hijacking
   - We can also notice that the hashes of the two wininet.dll files are non-equal, meaning the two DLLs themselves are completely different.   
    

