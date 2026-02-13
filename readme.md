# Azure Local Deployment Guide (v2508)

![Azure](https://img.shields.io/badge/Microsoft-Azure-blue)
![Version](https://img.shields.io/badge/Azure%20Local-2508-green)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success)
![License](https://img.shields.io/badge/License-MIT-yellow)

Community-driven deployment guide for Microsoft Azure Local environments integrated with Active Directory and Azure Arc.

---

## 📌 Overview

This guide provides a structured and production-ready walkthrough for deploying:

- Azure Local (v2508)
- Active Directory integration
- Azure Arc registration
- Azure Portal deployment
- Post-deployment validation

Official Microsoft documentation:
https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-introduction?view=azloc-2508

---

# 🏗 Architecture Summary

Azure Local integrates:

- On-premises physical servers
- Active Directory Domain Services
- Azure Resource Manager
- Azure Arc

This enables hybrid infrastructure management directly from Azure.

---

# 1️⃣ Prerequisites

## 1.1 Register Required Resource Provider

In Azure Portal:

Subscriptions → Resource Providers

Register:

```
Microsoft.Attestation
```

This step is mandatory for first-time deployments in the subscription.

---

# 2️⃣ Operating System Installation

## 2.1 Download ISO

Download the latest Azure Local ISO from Azure Portal.

## 2.2 Install on Physical Server

Mount ISO using:

- Dell iDRAC
- Lenovo XClarity

Install the OS (Windows Server–like installation process).

## 2.3 Initial Configuration

After first reboot:

- Set Administrator password (minimum 14 characters required)
- Configure:
  - Hostname
  - Static IP address
  - Default gateway
  - DNS (Domain Controller IP)
- Enable RDP access

---

# 3️⃣ Firmware and Driver Updates

⚠️ Mandatory before Azure Local deployment.

## Dell

Use:
- Dell System Update (DSU)

## Lenovo

Use:
- Lenovo Update Utility

Ensure all firmware, BIOS, NIC and storage drivers are updated.

---

# 4️⃣ Active Directory Preparation

## 4.1 Create Dedicated OU

Create a new Organizational Unit for Azure Local objects.

Example:

```
OU=AzureLocal,DC=company,DC=local
```

---

## 4.2 Run PowerShell on Domain Controller

```powershell
$password = ConvertTo-SecureString 'StrongPassword14+!' -AsPlainText -Force
$user = "azurehciuservs"

$credential = New-Object System.Management.Automation.PSCredential ($user, $password)

New-HciAdObjectsPreCreation `
-AzureStackLCMUserCredential $credential `
-AsHciOUName "OU=AzureLocal,DC=company,DC=local"
```

This will automatically create the required service account and AD objects.

---

# 5️⃣ Azure Arc Registration

Run on the Azure Local server:

```powershell
# Define variables
$Subscription = "<subscription-id>"
$RG = "RG-AzureLocal"
$Region = "eastus"   # use lowercase, no spaces
$Tenant = "<tenant-id>"

# Authenticate
Connect-AzAccount -SubscriptionId $Subscription -TenantId $Tenant -DeviceCode

# Retrieve tokens
$ARMtoken = (Get-AzAccessToken).Token
$id = (Get-AzContext).Account.Id

# Initialize Arc
Invoke-AzStackHciArcInitialization `
-SubscriptionID $Subscription `
-ResourceGroup $RG `
-TenantID $Tenant `
-Region $Region `
-Cloud "AzureCloud" `
-ArmAccessToken $ARMtoken `
-AccountID $id
```

Validate in:

Azure Portal → Azure Arc → Machines

Status must show: **Connected**

---

# 6️⃣ Deploy Azure Local Instance

In Azure Portal:

Search → Azure Local → Create → Azure Local Instance

## Important Configuration Notes

- Instance name must differ from hostname
- Select network model: `Compute_Storage`
- Custom Location must differ from:
  - Hostname
  - Instance name

Follow the wizard until deployment completes.

---

# 7️⃣ Post-Deployment

After successful deployment, you can configure:

- Virtual Machines
- AKS
- Azure Virtual Desktop
- Azure Site Recovery
- Backup and Monitoring integrations

---

# 🔐 Security Best Practices

- Use strong passwords (minimum 14 characters)
- Do not hardcode production credentials
- Use Azure Key Vault where applicable
- Apply least privilege access model
- Keep firmware and OS patched

---

# 🧪 Validation Checklist

✔ Resource Provider Registered  
✔ Firmware Updated  
✔ AD Objects Created  
✔ Azure Arc Connected  
✔ Azure Local Instance Deployed  

---

# 📁 Suggested Repository Structure

```
azure-local-deployment-guide/
│
├── README.md
├── CHANGELOG.md
├── LICENSE
└── docs/
```

---

# 🧾 CHANGELOG

## 1.0.0 – Initial Release
- Azure Local version 2508
- Full deployment workflow
- Active Directory integration
- Azure Arc onboarding
- Production-ready structure

---

# ⚠️ Disclaimer

This repository is community-maintained and is not an official Microsoft publication.

---

# 📜 License

MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files...

---

**Author:** Rodrigo Betussi  
**Last Update:** 16/09/2025  
