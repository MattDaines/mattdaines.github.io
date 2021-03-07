+++
author = "Matt Daines"
title = "Building DevTest Labs ARM Template Policies"
date = "2021-02-15"
description = "Deploying Azure DevTest Labs Policies with ARM Templates. An incomplete guide to some challenging Azure resources."
tags = [
    "Azure",
    "DevTest Labs",
    "ARM Templates"
]
categories = [
    "Azure",
    "IaC"
]
series = ["arm-templates"]
aliases = ["arm-templates"]
+++

I've recently been ARM templating again! In this article I'll go through how I've templated a relatively troublesome resource: `Microsoft.DevTestLab/labs/policysets/policies`. The documentation for this resource type is quite scarce so I figured I'd try figure out how to template at least some of the policies. Maybe you'll find something useful.
<!--more-->

### DevTest Labs Parent Resource

This resource is relatively simple; shouldn't have too much of an issue here! Not going into detail here but for completeness, this is what I used as my parent.

```json
{
  "name": "[parameters('devTestLabName')]",
  "type": "Microsoft.DevTestLab/labs",
  "apiVersion": "2018-09-15",
  "location": "[parameters('location')]",
  "properties": {
    "labStorageType": "Premium",
    "premiumDataDisks": "Enabled"
  }
}
```

### UserOwnedLabVmCount

This is where I first realised that threshold only accepts a string. My first assumption was that another field (factData) held strings and threshold held integers. That's not the case.

```json
{
  "name": "[concat(parameters('devTestLabName'), '/default/UserOwnedLabVmCount')]",
  "type": "Microsoft.DevTestLab/labs/policysets/policies",
  "apiVersion": "2018-09-15",
  "dependsOn": [
    "[resourceId('Microsoft.DevTestLab/labs', parameters('devTestLabName'))]"
  ],
  "properties": {
    "status": "Enabled",
    "factName": "UserOwnedLabVmCount",
    "threshold": "1",
    "evaluatorType": "MaxValuePolicy"
  }
}
```

There doesn't appear to be a max value for UserOwnedLabVmCount. I tried 999,999 and it took. If that's something you'd like to control in a parameter you can use a parameter of type int where you can set maxValue then for the threshold property use:

```json
"threshold": "[string(parameter('UserOwnedLabVmCountValue'))]"
```

### UserOwnedLabPremiumVmCount

Practically identical to the above!

```json
{
  "name": "[concat(parameters('devTestLabName'), '/default/UserOwnedLabPremiumVmCount')]",
  "type": "Microsoft.DevTestLab/labs/policysets/policies",
  "apiVersion": "2018-09-15",
  "dependsOn": [
    "[resourceId('Microsoft.DevTestLab/labs', parameters('devTestLabName'))]"
  ],
  "properties": {
    "status": "Enabled",
    "factName": "UserOwnedLabPremiumVmCount",
    "threshold": "1",
    "evaluatorType": "MaxValuePolicy"
  }
}
```

### LabVmCount

Another practically identical resource to the two previous policies... If only they were all this easy!

```json
{
  "name": "[concat(parameters('devTestLabName'), '/default/LabVmCount')]",
  "type": "Microsoft.DevTestLab/labs/policysets/policies",
  "apiVersion": "2018-09-15",
  "dependsOn": [
    "[resourceId('Microsoft.DevTestLab/labs', parameters('devTestLabName'))]"
  ],
  "properties": {
    "status": "Enabled",
    "factName": "LabVmCount",
    "threshold": "10",
    "evaluatorType": "MaxValuePolicy"
  }
}
```

### LabPremiumVmCount

And again... But this time a little caveat. This policy requires that the DevTest lab property `labStorageType` is set to `Premium`. ARM will throw a seemingly unrelated error if this is not set. I learnt this as initially my DevTest lab was set to StandardSSD.

```json
{
  "name": "[concat(parameters('devTestLabName'), '/default/LabPremiumVmCount')]",
  "type": "Microsoft.DevTestLab/labs/policysets/policies",
  "apiVersion": "2018-09-15",
  "dependsOn": [
    "[resourceId('Microsoft.DevTestLab/labs', parameters('devTestLabName'))]"
  ],
  "properties": {
    "status": "Enabled",
    "factName": "LabPremiumVmCount",
    "threshold": "10",
    "evaluatorType": "MaxValuePolicy"
  }
}
```

### LabVmSize

Okay, this one is different. Really. It's our first policy that doesn't just require a integer (.. as a string).

The solution for LabVmSize is to use a parameter of type array and parse that as a string. The array is just your standard Azure VM sizes in a list. To be verbose, this is the template resource component:

```json
{
  "name": "[concat(parameters('devTestLabName'), '/default/LabVmSize')]",
  "type": "Microsoft.DevTestLab/labs/policysets/policies",
  "apiVersion": "2018-09-15",
  "dependsOn": [
    "[resourceId('Microsoft.DevTestLab/labs', parameters('devTestLabName'))]"
  ],
  "properties": {
    "status": "Enabled",
    "factName": "LabVmSize",
    "threshold": "[string(parameters('allowedLabVmSizes'))]",
    "evaluatorType": "AllowedValuesPolicy"
  }
}
```

And the parameter component:

```json
"allowedLabVmSizes": {
  "value": [
    "Basic_A0",
    "Basic_A1",
    "Basic_A2",
    "Basic_A3",
    "Basic_A4"
  ]
}
```

### GalleryImage

This one beat me. I searched for `"/default/GalleryImage"` and found another blog that had managed to template this value. I wonder if they struggled as much as I did.

At first glance this policy is very similar to the one directly above. It accepts a JSON array as a string containing the usual details about the image reference. But for reasons that are beyond my comprehension they wouldn't stick. So, this is the template:

```json
{
  "name": "[concat(parameters('devTestLabName'), '/default/GalleryImage')]",
  "type": "Microsoft.DevTestLab/labs/policysets/policies",
  "apiVersion": "2018-09-15",
  "dependsOn": [
    "[resourceId('Microsoft.DevTestLab/labs', parameters('devTestLabName'))]"
  ],
  "properties": {
    "status": "Enabled",
    "factName": "GalleryImage",
    "threshold": "[concat('[', trim(parameters('allowedLabGalleryImages')), ']')]",
    "evaluatorType": "AllowedValuesPolicy"
  }
}
```

And as messy as it looks, this is how to provide a parameter file value to this resource.

```json
"allowedLabGalleryImages": {
    "value": "\"{\\\"offer\\\":\\\"WindowsServer\\\",\\\"publisher\\\":\\\"MicrosoftWindowsServer\\\",\\\"sku\\\":\\\"2019-Datacenter\\\",\\\"osType\\\":\\\"Windows\\\",\\\"version\\\":\\\"latest\\\"}\",\"{\\\"offer\\\":\\\"WindowsServer\\\",\\\"publisher\\\":\\\"MicrosoftWindowsServer\\\",\\\"sku\\\":\\\"2016-Datacenter\\\",\\\"osType\\\":\\\"Windows\\\",\\\"version\\\":\\\"latest\\\"}\""
  }
```

Not how I wanted it to look, at all. The above parameter value contains two image references that should help with adding additional values if you need to add more. It might help to break the value up onto multiple lines to help visualise what part you'll need to copy/paste.

### Unfinished Policies

Unfortunately, I've not managed to template for policies: `UserOwnedLabVmCountInSubnet`, `LabTargetCost`, `EnvironmentTemplate` and `ScheduleEditPermission`. I'll come back to them but just in case I don't in the near future (or, ever) I've listed some references that may or may not be helpful.

My approach to find the correct values for each policy has been to make the change in the Azure Portal then retrieve the Policy details back via PowerShell. I used a couple lines of script borrowed from the [DevTest Labs documentation](https://docs.microsoft.com/en-gb/azure/devtest-labs/scripts/set-allowed-vm-sizes-in-lab#sample-script)

```powershell
$lab = Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceName myDevTestLab

$labResourceName = $lab.Name + '/default'

$existingPolicy = (Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs/policySets/policies' -ResourceName $labResourceName -ResourceGroupName $lab.ResourceGroupName -ApiVersion 2016-05-15)
```

You can update $existingPolicy whenever you make a change to the policies.

`$existingPolicy.Count` - will return the number of configured policy resources.
`$existingPolicy[0]` - will return the first policy configured on the DevTest lab.
`$existingPolicy[0].Properties` - will return the properties of a configured policy. It's this information that you'll need to configure policies.
In the case of at least GalleyImage you might find it easier to read if you run `$existingPolicy[0].Properties.threshold | ConvertFrom-Json`

### References

[ARM Docs: DevTest Labs](https://docs.microsoft.com/en-gb/azure/templates/microsoft.devtestlab/labs)

[ARM Docs: DevTest Labs Policies](https://docs.microsoft.com/en-gb/azure/templates/microsoft.devtestlab/labs/policysets/policies)

[GalleryImage](https://integrationworkz.antiohne.nl/2018/11/deploy-with-arm-templates-azure-devtest.html) - The blog that helped me solve GalleryImage.
