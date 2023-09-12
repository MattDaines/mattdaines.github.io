---
title: Enabling/Disabling Network Level Authentication (NLA)
description: How to authenticate as a Service Principal in Azure with PowerShell
date: 2023-05-09 00:00:00+0000
#image: header-image.png
categories: [
    "Windows",
    "Azure"
]
---

> The remote computer that you are trying to connect to requires Network Level Authentication (NLA), but your Windows domain controller cannot be contacted to perform NLA. If you are an administrator on the remote computer, you can disable NLA by using the options on the Remote tab of the System Properties dialog box.

A frustrating error that you may occasionally bump into while configuring Windows server infrastructure. It's likely that a networking issue is preventing the member server from contacting the domain controller. Even after the networking issue is resolved, a domain admin (or, another privileged admin) would need to re-establish the trust to the domain controller from the member server.

My, admittedly somewhat limited knowledge of the issue of why this happens is when the domain member tries to reset it's own password/credential and can't due to a network issue (I'm sure there's other reason, but that's the most common from my personal experience). The machine password expires and then the member server cannot authenticate to the domain, which is part of the authentication process for domain accounts when NLA is enabled.

This page will list a few PowerShell cmdlets that I run when I'm happy that the member server can be brought back. Just don't forget to re-enable NLA once you're the computer is trusted again.

## Disable Network Level Authentication (NLA)

In Azure, you likely need to run this from the Run Command blade on the target virtual machine.

```powershell
(Get-WmiObject -class Win32_TSGeneralSetting -Namespace root\cimv2\terminalservices -ComputerName (hostname) -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0)
```

You should now be able to remote on with the local administrator account. If you're not sure what the local administrator account is, you can use the Reset Password blade in the Azure portal - this creates a new local admin account if it does not already exist or resets the password if it does.

## Domain Trust

If you successfully remote onto the virtual machine with the local admin credential, you can now reset the computer password and re-establish the trust with the domain.

For `<Domain>\<User>` you'll want to use the domain account, not the local account. You'll be prompted for the password.

```powershell
Reset-ComputerMachinePassword -Credential <Domain>\<User>
```

## Enable Network Level Authentication (NLA)

Assuming the `Reset-ComputerMachinePassword` succeeded, you can now re-enable NLA and log back on with your domain account.

```powershell
(Get-WmiObject -class Win32_TSGeneralSetting -Namespace root\cimv2\terminalservices -ComputerName (hostname) -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(1)
```
