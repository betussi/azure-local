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

Azure Local is full-stack infrastructure software that runs on OEM-validated bare-metal hardware to support virtual machines, containers, and select Azure services with central cloud management enabled via Azure Arc. :contentReference[oaicite:1]{index=1}

---

## 📦 What’s New in Version 2601

The January 2026 2601 release includes:

- OS version update: Azure Local nodes now run OS build **26100.32230**. :contentReference[oaicite:2]{index=2}  
- **VM Connect (Preview):** Connect to VMs with no network or boot failures. :contentReference[oaicite:3]{index=3}  
- **Rack aware clustering (GA):** Local availability zones at rack level. :contentReference[oaicite:4]{index=4}  
- **Diagnostics log collection** from Azure Portal. :contentReference[oaicite:5]{index=5}  
- **Drift detection framework** for component state validation. :contentReference[oaicite:6]{index=6}  
- Security baseline updates and pre-upgrade validation enhancements. :contentReference[oaicite:7]{index=7}

---

## 🏗 Architecture Overview

Azure Local integrates:

- On-premises physical servers
- Active Directory Domain Services
- Azure Resource Manager
- Azure Arc

This enables hybrid infrastructure management directly from Azure.

---

## 1️⃣ Prerequisites

### Register Required Resource Provider

Azure Portal → **Subscriptions** → **Resource Providers**

Register:

```
Microsoft.Attestation
```

This step is required before first deployment.

---

## 2️⃣ Operating System Installation

### Download ISO

Download the Azure Local ISO from Azure Portal (OS build **26100.32230** or later).

### Install on Physical Server

Mount ISO using:

- iDRAC (Dell)
- XClarity (Lenovo)

Configure:

- Administrator password (min 14 characters)
- Hostname
- Static IP
- Gateway
- DNS (Domain Controller)
- Enable RDP

---

## 3️⃣ Firmware & Driver Updates

⚠️ Mandatory before deployment.

### Dell

Use Dell System Update (DSU)

### Lenovo

Use Lenovo Update Utility

Ensure firmware, NIC and storage drivers are up-to-date.

---

## 4️⃣ Active Directory Preparation

### Create Organizational Unit

Example:

```
OU=AzureLocal,DC=company,DC=local
```

### Create Deployment Service Account

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

## 5️⃣ Azure Arc Registration

Run on Azure Local server:

```powershell
# Variables
$Subscription = "<subscription-id>"
$RG = "RG-AzureLocal"
$Region = "eastus"
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

Validate in Azure Portal → Azure Arc → Machines.

Status must show **Connected**.

---

## 6️⃣ Deploy Azure Local Instance

In Azure Portal:

**Search** → Azure Local → **Create** → Azure Local Instance

### Important Notes

- Instance name must differ from hostname
- Network model: `Compute_Storage`
- Custom Location must differ from hostname and instance name

Follow the wizard until provisioning completes.

---

## 7️⃣ Post-Deployment

After deployment, you can configure and manage:

- Virtual Machines
- AKS
- Azure Virtual Desktop
- Site Recovery
- Monitoring & Diagnostics

---

## 🔍 Known Issues & Considerations

Review the latest known issues before deployment — these include important considerations for Azure Local releases. :contentReference[oaicite:8]{index=8}

---

## 🔐 Security Best Practices

- Use strong passwords (min 14 characters)
- Avoid hardcoding credentials
- Use Azure Key Vault where possible
- Apply least privileged access
- Keep OS and firmware patched

---

## 🧾 CHANGELOG

### 1.0.1 – Updated for 2601
- Added 2601 new features section
- Updated OS build references and requirements
- Known issues section

---

## ⚠️ Disclaimer

This repository is community-maintained and not an official Microsoft publication.

---

## 📜 License

MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files...
