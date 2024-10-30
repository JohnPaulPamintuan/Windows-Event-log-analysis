# Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server

## Description:
- <b>Windows Log Event</b>:  are an intrinsic part of the Windows Operating System, storing logs from different components of the system including the system itself, applications running on it, ETW providers, services, and others. Windows event logging offers comprehensive logging capabilities for application errors, security events, and diagnostic information.

## Tool Used: 
- <b>Event logs </b>: Built-in logging mechanism in Windows for recording system events, including security events, application events, and system events.
- <b>Event Viewer</b> : A Microsoft Management Console (MMC) application that allows users to view and analyze event logs on Windows systems.
- <b>Sysmon (System Monitor) </b> : A Windows system service that logs system activity to the Windows Event Log.

## Purposes: 
- Enhance security visibility on Windows Servers.
- Detect malicious activities like DLL hijacking.

# Setting Up the Environment: 
## <b>Accessing Event Viewer</b>:
- Open Event Viewer

     <img src="https://i.imgur.com/mUTcNU7.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

- Familiarize yourself with the main log categories:
  - Application Logs: Logs related to applications and services.
  - Security Logs: Logs related to security events (e.g., logon attempts).
  - System Logs: Logs related to Windows system events.

      <img src="https://i.imgur.com/cs76zjJ.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
### Key Event Logs to Monitor
- <b><ins>Security Log:</b></ins>
  - Event ID 4624: Successful logon.
  - Event ID 4625: Failed logon attempts (indicating potential brute force attacks).
  - Event ID 4648: A logon attempt using explicit credentials.
- <b><ins>System Log:</b></ins>
  - Event ID 6005: The Event Log service was started (system boot).
  - Event ID 6006: The Event Log service was stopped (system shutdown).
- <b><ins>Application Log:</b></ins>
  - Monitor for application errors or warnings that may indicate malicious activity.

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
    <img src="https://i.imgur.com/8xMkiWC.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    <img src="https://i.imgur.com/KvvJRgJ.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    

### Collecting Event Logs
- Open Event Viewer and navigate to <b>Applications and Services Logs > Microsoft > Windows > Sysmon > Operational</b>
    <img src="https://i.imgur.com/RtH4E85.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>

### Identifying Important Events:
- Look for:
  - Event ID 1: Process creation.
  - Event ID 7: Image loaded (for DLL loading).
  - Event ID 10: Process accessed.

    <img src="https://i.imgur.com/0Sh2MU9.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    <img src="https://i.imgur.com/tKxRbgf.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    <img src="https://i.imgur.com/HNzN7o1.png" height="60%" width="80%" alt="Using Sysmon and Event logs to detect and analyse malicious activity on Windows Server"/>
    
    
     
