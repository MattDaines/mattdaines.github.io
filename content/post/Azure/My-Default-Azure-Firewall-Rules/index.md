---
title: My Default Azure Firewall Rules
description: How to authenticate as a Service Principal in Azure with PowerShell
date: 2023-09-13 00:00:00+0000
#image: header-image.png
categories: [
    "Azure"
]
---

Typically, in a Cloud Adoption Framework Landing Zone you will use an Azure Firewall to filter traffic heading to the Internet. If you are using a standard tier Azure Firewall or Firewall Policy you will not be able to use web categories, so you will need to create a rule for each FQDN that you want to allow. It can be a pain to find all the FQDNs that you need to allow, so I have created a list of the FQDNs that I typically allow as a starting point. I'll then look through the firewall logs to see if there are any other FQDNs that I need to allow.

> I'm going to add to this page as I discover more endpoints that I need to allow.

## Public Key Infrastructure (PKI)

### OSCPs

| FQDN | Protocol | Port | Description | 
| - | - | - | - |
| ocsp.digicert.com | TCP | 80 | OCSP for DigiCert |
| ts-ocsp.ws.symantec.com | TCP | 80 | OCSP for Symantec |
| ocsp.comodoca.com | TCP | 80 | OCSP for Comodo |
| oneocsp.microsoft.com | TCP | 80 | OCSP for Microsoft |

### CRL

| FQDN | Protocol | Port | Description | 
| - | - | - | - |
| crl3.digicert.com, crl4.digicert.com | TCP | 80 | CRL for DigiCert |
| crl.verisign.com | TCP | 80 |  CRL for Symantec |
| crl.comodoca.com | TCP | 80 | CRL for Comodo |

## Azure Defaults

| FQDN | Protocol | Port | Description | 
| - | - | - | - |
| time.windows.com | UDP | 123 | Default time server for Windows based OS |
| azkms.core.windows.net, kms.core.windows.net | TCO | 1688 | KMS server for Windows based OS. [Source 1 - Annoucement](https://azure.microsoft.com/en-gb/updates/new-kms-dns-in-azure-global-cloud/) [Source 2 - Activation Troubleshooting](https://learn.microsoft.com/en-gb/troubleshoot/azure/virtual-machines/custom-routes-enable-kms-activation#solution) |
