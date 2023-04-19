# Authentication to Azure using PowerShell

Hmm, something.

<!--more-->
How to authenticate to Azure, apparently.

## Service Principal

```powershell
$Credential = New-Object -TypeName System.Management.Automation.PSCredential `
	-ArgumentList $ClientId, ($ClientPassWord | ConvertTo-SecureString -AsPlainText)

Connect-AzAccount -ServicePrincipal -TenantId $TenantId -Credential $Credential
```