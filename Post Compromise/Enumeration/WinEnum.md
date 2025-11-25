# Windows Enumeration
## Enumerating MS Windows host, for enumerating MS Active Directory

### System
first get information about the system we are in

```cmd
C:\> systeminfo

Host Name:                 WIN-SERVER-CLI
OS Name:                   Microsoft Windows Server 2022 Standard
OS Version:                10.0.20348 N/A Build 20348
OS Manufacturer:           Microsoft Corporation
[...]
Hotfix(s):                 3 Hotfix(s) Installed.
                           [01]: KB5013630
[...]
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
[...]
```

### Check Installed update 
using `wmic qfe get Caption,Description`, for knowing how quickly systems are being patched and updated

```cmd
C:\> wmic qfe get Caption,Description
Caption                                     Description      
http://support.microsoft.com/?kbid=5013740  Update
https://support.microsoft.com/help/5018494  Security Update
                                            Update
```

### Looking for app installed version
using `wmic product get name,version`

```cmd
C:\> wmic product get name,version,vendor
Name                                                            Vendor                                   Version
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29910     Microsoft Corporation                    14.28.29123
[...]
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29910  Microsoft Corporation                    14.28.29123
```
>Important

>The WMI command-line (WMIC) utility is deprecated as of Windows 10, version 21H1, and as of the 21H1 semi-annual channel release of Windows Server.
>This utility is superseded by Windows PowerShell for WMI (see Chapter 7 - Working with WMI).
>This deprecation applies only to the WMI command-line (WMIC) utility; Windows Management Instrumentation (WMI) itself is not affected.
>Also see Windows 10 features we're no longer developing.


### Check what service is installed and started
using 	`net start`

```cmd
C:\> net start
These Windows services are started:

   Base Filtering Engine
   Certificate Propagation
   Client License Service (ClipSVC)
   COM+ Event System
   Connected User Experiences and Telemetry
   CoreMessaging
   Cryptographic Services
   DCOM Server Process Launcher
   DHCP Client
   DNS Client
[...]
   Windows Time
   Windows Update
   WinHTTP Web Proxy Auto-Discovery Service
   Workstation

The command completed successfully.
```

### Knowing who you are and what privilage do you have and which group you're in

```cmd
C:\> whoami
win-server-cli\0xbscure

C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
[...]
```

using `whoami /priv` for enumerating Privileges, `whoami /groups` for knowing which group you belong to

### Viewing users by running `net user`

```cmd
C:\> net user

User accounts for \\WIN-SERVER-CLI

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
0xbscure                 peter                    cumit
WDAGUtilityAccount
The command completed successfully.
```

### Discover the available groups using `net group` if the system is domain controller, or `net localgroup` otherwise

```cmd
C:\> net localgroup

Aliases for \\WIN-SERVER-CLI

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Administrators
*Backup Operators
*Certificate Service DCOM Access
*Cryptographic Operators
*Device Owners
[...]
```

### List the users that belong the group i,e administrator group using `net localgroup administrator`

```cmd
C:\> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
0xbscure
peter
cumit
The command completed successfully.
```

Use `net accounts` to see the local settings on a machine, use `net accounts /domain` if the machine belongs to a domain

```cmd
C:\> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          0
Maximum password age (days):                          42
Minimum password length:                              0
Length of password history maintained:                None
Lockout threshold:                                    10
Lockout duration (minutes):                           10
Lockout observation window (minutes):                 10
Computer role:                                        WORKSTATION
The command completed successfully.
```



