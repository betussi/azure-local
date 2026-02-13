# Azure Local Deployment Guide (v2601)

![Azure](https://img.shields.io/badge/Microsoft-Azure-blue)
![Version](https://img.shields.io/badge/Azure%20Local-2601-green)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success)
![License](https://img.shields.io/badge/License-MIT-yellow)

Comprehensive deployment guide for Microsoft Azure Local environments integrated with Active Directory and Azure Arc.

Official documentation:
https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-introduction?view=azloc-2601

---

## 📌 Overview

Azure Local is full-stack infrastructure software that runs on OEM-validated bare-metal hardware to support virtual machines, containers, and select Azure services with centralized cloud management enabled via Azure Arc.

This guide provides a structured and production-ready walkthrough for deploying:

- Azure Local (v2601)
- Active Directory integration
- Azure Arc registration
- Azure Portal deployment
- Post-deployment validation

---

## 📦 What’s New in Version 2601

The 2601 release includes:

- Updated OS build
- VM Connect (Preview)
- Rack-aware clustering (GA)
- Diagnostics log collection from Azure Portal
- Drift detection framework
- Security baseline improvements
- Enhanced pre-upgrade validation checks

---

# 🏗 Architecture Overview

Azure Local integrates:

- On-premises physical servers
- Active Directory Domain Services
- Azure Resource Manager
- Azure Arc

This enables hybrid infrastructure management directly from Azure.

---

# 1️⃣ Prerequisites

## 1.1 Register Required Resource Provider

Azure Portal → **Subscriptions** → **Resource Providers**

Register:

```
Microsoft.Attestation
```

This step is required before first deployment.

---

# 2️⃣ Operating System Installation

## 2.1 Download ISO

Download the Azure Local ISO from Azure Portal (latest build available).

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

# 3️⃣ Firmware & Driver Updates

⚠️ Mandatory before deployment.

## Dell

Use Dell System Update (DSU)

## Lenovo

Use Lenovo Update Utility

Ensure firmware, BIOS, NIC and storage drivers are updated.

---

# 4️⃣ Active Directory Preparation

## 4.1 Create Dedicated OU

Create a new Organizational Unit for Azure Local objects.

Example:

```
OU=AzureLocal,DC=company,DC=local
```

---

## 4.2 Create Deployment Service Account

Run on the Domain Controller:

```powershell
$password = ConvertTo-SecureString 'StrongPassword14+!' -AsPlainText -Force
$user = "azurehciuservs"

$credential = New-Object System.Management.Automation.PSCredential ($user, $password)

New-HciAdObjectsPreCreation `
-AzureStackLCMUserCredential $credential `
-AsHciOUName "OU=AzureLocal,DC=company,DC=local"
```

This creates the required service account and AD objects.

---

# 5️⃣ Azure Arc Registration

Run on the Azure Local server:

```powershell
# Variables
$Subscription = "<subscription-id>"
$RG = "RG-AzureLocal"
$Region = "eastus"   # lowercase, no spaces
$Tenant = "<tenant-id>"

Connect-AzAccount -SubscriptionId $Subscription -TenantId $Tenant -DeviceCode

$ARMtoken = (Get-AzAccessToken).Token
$id = (Get-AzContext).Account.Id

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

# 🧾 CHANGELOG

## 1.0.1 – Updated for 2601
- Documentation updated to latest version
- New features section added
- Security and validation improvements

---

# ⚠️ Disclaimer

This repository is community-maintained and is not an official Microsoft publication.

---

# 📜 License

MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files...

# 👨‍💻 Author

**Rodrigo Felipe Betussi**  
Cloud & Infrastructure Specialist  
Microsoft Certified Professional  
Last update: January 2026
