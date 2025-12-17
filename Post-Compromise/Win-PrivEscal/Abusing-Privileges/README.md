# Abusing Dangerous Privileges

to show privileges use command `whoami /priv` to see privileges of the current user

## SeBackup / SeRestore Privileges
These privileges allow bypassing DACLs to read/write any file, useful for backup operations.

`SeRestorePrivilege` allows attackers to write to protected directories and overwrite trusted binaries.

Attackers can use these privileges to copy registry hives `SAM and SYSTEM` and extract password hashes. Tools like `Impacket’s secretsdump` can be used to retrieve the hashes and then perform `Pass-the-Hash` attacks to gain SYSTEM access.

##SeTakeOwnership Privilege
Allows users to take ownership of any file or object on the system.

An attacker can use this privilege to replace system executables `(like utilman.exe)`, giving themselves `SYSTEM-level access`. The process includes taking ownership of the file, granting full permissions, and replacing the executable with a payload.

## SeImpersonate / SeAssignPrimaryToken Privileges
These privileges allow a process to impersonate another user, enabling it to act on their behalf.

Attackers can exploit these privileges by compromising services like IIS, which use accounts with impersonation privileges (e.g., LOCAL SERVICE, NETWORK SERVICE). Using tools like `RogueWinRM`, an attacker can spawn a process that impersonates a privileged user (e.g., SYSTEM) and execute commands remotely via a malicious connection.

## SeAudit Privileges
`SeAuditPrivilege` allows a user to generate security audit events, meaning they can write entries to the Security Event Log

`SeAuditPrivilege` does not directly clear logs, it enables:

    Log manipulation

    Log pollution

    Creation of misleading security events

This supports `anti-forensics` without triggering high-risk log deletion alerts.

`SeAuditPrivilege` enables attackers to undermine detection mechanisms by Injecting benign or misleading security events, Obscuring attacker activity, Disrupting SOC visibility and correlation logic

## SeCreatePagefile Privilege
Create hiberfil.sys, read it offline, look for sensitive data.

## SeCreateToken Privilege
Create arbitrary token including local admin rights with NtCreateToken.

## SeLoadDriver Privilege
1. Load buggy kernel driver such as szkg64.sys
2. Exploit the driver vulnerability

Alternatively, the privilege may be used to unload security-related drivers with fltMC builtin command. i.e.: fltMC sysmondrv

## SeManageVolume Privilege
1. Enable the privilege in the token
2. Create handle to `\.\C:` with `SYNCHRONIZE | FILE_TRAVERSE`
3. Send the `FSCTL_SD_GLOBAL_CHANGE` to replace `S-1-5-32-544` with `S-1-5-32-545`
4. Overwrite utilman.exe etc.

`FSCTL_SD_GLOBAL_CHANGE` can be made with this [piece of code](https://github.com/gtworek/PSBits/blob/master/Misc/FSCTL_SD_GLOBAL_CHANGE.c)

## SeRelabel Privilege
Modification of system files by a legitimate administrator

Integrity labels provide additional protection, on top of well-known ACLs. Two main scenarios include:
- protection against attacks using exploitable applications such as browsers, PDF readers etc.
- protection of OS files.

SeRelabel present in the token will allow to use `WRITE_OWNER` access to a resource, including files and folders. Unfortunately, the token with IL less than High will have SeRelabel privilege disabled, making it useless for anyone not being an admin already.
