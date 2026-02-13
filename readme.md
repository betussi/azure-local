# Azure Local Deployment Guide | Guia de Implementação Azure Local (v2601)

![Azure](https://img.shields.io/badge/Microsoft-Azure-blue)
![Version](https://img.shields.io/badge/Azure%20Local-2601-green)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success)
![License](https://img.shields.io/badge/License-MIT-yellow)

🌎 **Bilingual Documentation (English + Português Brasil)**  
🌎 **Documentação Bilíngue (English + Português Brasil)**  

Official Microsoft documentation / Documentação oficial:  
https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-introduction?view=azloc-2601

---

# 🇧🇷 Versão em Português (PT-BR)

## 📌 Visão Geral

O Azure Local é uma solução de infraestrutura hiperconvergente executada em hardware validado por fabricantes (OEM), permitindo a execução de:

- Máquinas Virtuais
- Containers
- Serviços híbridos do Azure

A gestão é centralizada via Azure através do Azure Arc.

---

## 📦 Novidades da Versão 2601

- Atualização da build do sistema operacional
- VM Connect (Preview)
- Cluster com reconhecimento de rack (GA)
- Coleta de logs via Portal do Azure
- Framework de detecção de drift
- Melhorias no baseline de segurança

---

## 🏗 Arquitetura

Integração entre:

- Servidores físicos on-premises
- Active Directory Domain Services
- Azure Resource Manager
- Azure Arc

---

## 1️⃣ Pré-requisitos

Registrar o provider:

```
Microsoft.Attestation
```

Portal do Azure → Assinaturas → Resource Providers

---

## 2️⃣ Instalação do Sistema Operacional

- Baixar ISO no Portal
- Montar via iDRAC (Dell) ou XClarity (Lenovo)
- Instalar como Windows Server
- Configurar:
  - Senha (mín. 14 caracteres)
  - Hostname
  - IP estático
  - DNS (Domain Controller)
  - Habilitar RDP

---

## 3️⃣ Atualização de Firmware

Obrigatório antes do deploy.

- Dell → DSU
- Lenovo → Lenovo Update Utility

---

## 4️⃣ Preparação do Active Directory

Criar OU:

```
OU=AzureLocal,DC=empresa,DC=local
```

Executar no DC:

```powershell
$password = ConvertTo-SecureString 'SenhaForteCom14+!' -AsPlainText -Force
$user = "azurehciuservs"

$credential = New-Object System.Management.Automation.PSCredential ($user, $password)

New-HciAdObjectsPreCreation `
-AzureStackLCMUserCredential $credential `
-AsHciOUName "OU=AzureLocal,DC=empresa,DC=local"
```

---

## 5️⃣ Registro no Azure Arc

```powershell
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

Validar: Azure Arc → Máquinas → Status **Connected**

---

## 6️⃣ Deploy da Instância

Portal → Azure Local → Criar

⚠️ Nome da instância ≠ Hostname  
⚠️ Modelo de rede: `Compute_Storage`

---

## 🔐 Boas Práticas

- Senhas fortes
- Menor privilégio
- Atualizações aplicadas
- Evitar credenciais em texto plano

---

# 🇺🇸 English Version

## 📌 Overview

Azure Local is a hyperconverged infrastructure solution running on OEM-validated hardware, enabling:

- Virtual Machines
- Containers
- Hybrid Azure services

Centralized management is enabled via Azure Arc.

---

## 📦 What’s New in 2601

- OS build update
- VM Connect (Preview)
- Rack-aware clustering (GA)
- Portal-based diagnostics collection
- Drift detection framework
- Security baseline improvements

---

## 🏗 Architecture

Integration between:

- On-premises physical servers
- Active Directory Domain Services
- Azure Resource Manager
- Azure Arc

---

## 1️⃣ Prerequisites

Register provider:

```
Microsoft.Attestation
```

Azure Portal → Subscriptions → Resource Providers

---

## 2️⃣ Operating System Installation

- Download ISO from Azure Portal
- Mount via iDRAC (Dell) or XClarity (Lenovo)
- Install OS
- Configure:
  - Password (min 14 characters)
  - Hostname
  - Static IP
  - DNS (Domain Controller)
  - Enable RDP

---

## 3️⃣ Firmware Updates

Mandatory before deployment.

- Dell → DSU
- Lenovo → Lenovo Update Utility

---

## 4️⃣ Active Directory Preparation

Create OU:

```
OU=AzureLocal,DC=company,DC=local
```

Run on Domain Controller:

```powershell
$password = ConvertTo-SecureString 'StrongPassword14+!' -AsPlainText -Force
$user = "azurehciuservs"

$credential = New-Object System.Management.Automation.PSCredential ($user, $password)

New-HciAdObjectsPreCreation `
-AzureStackLCMUserCredential $credential `
-AsHciOUName "OU=AzureLocal,DC=company,DC=local"
```

---

## 5️⃣ Azure Arc Registration

```powershell
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

Validate in Azure Arc → Machines → Status **Connected**

---

## 🔐 Security Best Practices

- Strong passwords
- Least privilege model
- Keep firmware and OS updated
- Avoid hardcoded credentials

---

# ⚠️ Disclaimer | Aviso

This repository is community-maintained and is not an official Microsoft publication.  
Este repositório é mantido pela comunidade e não é uma publicação oficial da Microsoft.

---

# 👨‍💻 Author | Autor

**Rodrigo Felipe Betussi**  
Cloud & Infrastructure Specialist  
Microsoft Certified Professional  
Last update / Última atualização: January 2026
