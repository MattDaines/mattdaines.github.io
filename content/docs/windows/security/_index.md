---
bookCollapseSection: true
---

# Windows - Network Level Authentication (NLA)

Sometimes, you need to disable NLA on a Windows domain member machine to allow remote access and resolve domain trust issues. The NLA script below is how to disable and enable NLA. You should re-enable NLA once the machine is trusted with the domain.

## Disable NLA
```powershell
(Get-WmiObject -class Win32_TSGeneralSetting -Namespace root\cimv2\terminalservices -ComputerName (hostname) -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0)
```

You should now be able to remote on with the local administrator account (unless it's a domain controller).

## Domain Trust

```powershell
Reset-ComputerMachinePassword -Credential <Domain>\<User>
```

## Enable NLA

```powershell
(Get-WmiObject -class Win32_TSGeneralSetting -Namespace root\cimv2\terminalservices -ComputerName (hostname) -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(1)
```

{{<section>}}
