<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:1F9BD4,50:2E75B6,100:16265F&height=200&section=header&text=resumo-do-lab&fontSize=52&fontColor=FFFFFF&fontAlignY=38&desc=Guia%20Abrangente%20de%20Microsoft%20Azure&descAlignY=58&descSize=20&animation=fadeIn" width="100%"/>

[![Azure](https://img.shields.io/badge/Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com)
[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io)
[![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)](https://learn.microsoft.com/en-us/powershell/)
[![DevOps](https://img.shields.io/badge/Azure%20DevOps-0078D7?style=for-the-badge&logo=azuredevops&logoColor=white)](https://dev.azure.com)

[![Status](https://img.shields.io/badge/status-ativo-brightgreen?style=flat-square)]()
[![Idioma](https://img.shields.io/badge/idioma-pt--BR-009C3B?style=flat-square)]()
[![Nível](https://img.shields.io/badge/nível-intermediário%20→%20avançado-2E75B6?style=flat-square)]()
[![Licença](https://img.shields.io/badge/licença-MIT-blue?style=flat-square)]()

</div>

---

## 📖 Sobre o Projeto

O **resumo-do-lab** é um repositório abrangente de aprendizado e referência técnica para **Microsoft Azure**, cobrindo desde fundamentos de nuvem até práticas avançadas de **DevOps**, **governança**, **segurança** e **infraestrutura como código**. Ideal para profissionais que estão se preparando para certificações Azure ou buscam consolidar conhecimentos práticos.

> 🎯 **Objetivo:** Concentrar em um único lugar os conceitos, exemplos práticos e artefatos utilizados no dia a dia de um engenheiro de nuvem Azure.

---

## 🗂️ Estrutura do Repositório

```
resumo-do-lab/
├── 📄 README.md
├── 🏗️ terraform/
│   └── main.tf                  # Infraestrutura como código (IaC)
├── ⚙️ azure-pipelines.yml        # Pipeline CI/CD no Azure DevOps
└── 📜 scripts/
    └── deploy.ps1               # Script PowerShell de implantação
```

---

## ☁️ Modelos de Serviço Cloud

<div align="center">

| Modelo | Nome Completo | Responsabilidade | Exemplos Azure |
|--------|--------------|-----------------|---------------|
| 🏗️ **IaaS** | Infrastructure as a Service | Usuário gerencia SO, runtime, apps | Azure VMs, Discos, VNets |
| 🧩 **PaaS** | Platform as a Service | Usuário gerencia apenas aplicação e dados | App Service, Azure SQL, AKS |
| ☁️ **SaaS** | Software as a Service | Provedor gerencia tudo | Microsoft 365, Dynamics 365 |
| ⚡ **FaaS** | Function as a Service | Execução serverless orientada a eventos | Azure Functions, Logic Apps |

</div>

---

## 🌍 Infraestrutura Global Azure

<details>
<summary>🗺️ Regiões e Zonas de Disponibilidade</summary>

```
Azure Global Infrastructure
│
├── 🌎 Geografias (Geographies)
│   ├── Conjunto de regiões com requisitos legais/compliance comuns
│   └── Ex: Brasil, Europa, América do Norte, Ásia-Pacífico
│
├── 🏙️ Regiões (Regions)
│   ├── Localização geográfica específica
│   ├── Ex: Brazil South (São Paulo), East US, West Europe
│   └── Par de regiões para DR e continuidade de negócios
│
└── 🏢 Zonas de Disponibilidade (Availability Zones)
    ├── 3+ datacenters independentes por região
    ├── Energia, rede e refrigeração isoladas
    └── SLA 99,99% para VMs em múltiplas zonas
```

</details>

---

## 🏗️ Infraestrutura como Código

<details>
<summary>📋 Terraform — Exemplo <code>main.tf</code></summary>

```hcl
# terraform/main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "sttfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location
  tags     = local.common_tags
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "vnet-${var.environment}-${var.location_short}"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

# Storage Account
resource "azurerm_storage_account" "main" {
  name                     = "st${var.project}${var.environment}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "GRS"
  min_tls_version          = "TLS1_2"
}
```

</details>

<details>
<summary>⚙️ Azure Pipelines — CI/CD YAML</summary>

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  terraformVersion: '1.5.0'
  azureSubscription: 'AzureServiceConnection'

stages:
  - stage: Validate
    displayName: '🔍 Validação'
    jobs:
      - job: TerraformValidate
        steps:
          - task: TerraformInstaller@1
            inputs:
              terraformVersion: $(terraformVersion)
          - script: terraform init -backend=false
            displayName: 'Terraform Init'
          - script: terraform validate
            displayName: 'Terraform Validate'
          - script: terraform fmt -check
            displayName: 'Terraform Format Check'

  - stage: Plan
    displayName: '📋 Plano'
    dependsOn: Validate
    jobs:
      - job: TerraformPlan
        steps:
          - task: TerraformTaskV4@4
            inputs:
              provider: 'azurerm'
              command: 'plan'
              environmentServiceNameAzureRM: $(azureSubscription)

  - stage: Apply
    displayName: '🚀 Deploy'
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: TerraformApply
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: TerraformTaskV4@4
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    environmentServiceNameAzureRM: $(azureSubscription)
```

</details>

<details>
<summary>📜 PowerShell — Script de Deploy</summary>

```powershell
# scripts/deploy.ps1
[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [string]$ResourceGroupName,
    [Parameter(Mandatory=$true)]
    [string]$Location,
    [string]$Environment = "dev"
)

# Conectar ao Azure
Connect-AzAccount -Identity

# Verificar/Criar Resource Group
$rg = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
if (-not $rg) {
    Write-Host "Criando Resource Group: $ResourceGroupName" -ForegroundColor Cyan
    New-AzResourceGroup -Name $ResourceGroupName -Location $Location
}

# Deploy ARM Template
$deployParams = @{
    ResourceGroupName = $ResourceGroupName
    TemplateFile      = "./arm-template.json"
    TemplateParameterObject = @{
        environment = $Environment
        location    = $Location
    }
}

New-AzResourceGroupDeployment @deployParams -Verbose
Write-Host "Deploy concluído com sucesso!" -ForegroundColor Green
```

</details>

---

## 🔐 Segurança e Identidade

<div align="center">

[![Azure AD](https://img.shields.io/badge/Azure%20AD-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)]()
[![IAM](https://img.shields.io/badge/IAM%20%2F%20RBAC-16265F?style=flat-square)]()
[![PIM](https://img.shields.io/badge/PIM-2E75B6?style=flat-square)]()
[![Purview](https://img.shields.io/badge/Microsoft%20Purview-0078D4?style=flat-square)]()

</div>

| Recurso | Descrição |
|---------|-----------|
| **IAM / RBAC** | Controle de acesso baseado em funções — Owner, Contributor, Reader, roles customizadas |
| **PIM** | Privileged Identity Management — acesso just-in-time com aprovação e auditoria |
| **JIT Access** | Acesso temporário a VMs — reduz superfície de ataque em portas de gerenciamento |
| **Microsoft Purview** | Governança de dados, catalogação, classificação e proteção de informações sensíveis |
| **Azure Policy** | Garante conformidade de recursos com regras organizacionais |

---

## 🌐 Redes e Conectividade

<details>
<summary>🔌 VNet, NSG e Firewall</summary>

```
Arquitetura de Rede Hub-and-Spoke
┌─────────────────────────────────────────────┐
│              Hub VNet (10.0.0.0/16)         │
│  ┌─────────────┐    ┌────────────────────┐  │
│  │ Azure       │    │  Azure Bastion     │  │
│  │ Firewall    │    │  (acesso seguro)   │  │
│  └─────────────┘    └────────────────────┘  │
│         │                    │               │
│  ┌──────▼────────────────────▼──────────┐   │
│  │        VNet Peering (bi-direcional)  │   │
│  └──────┬────────────────────┬──────────┘   │
└─────────┼────────────────────┼──────────────┘
          │                    │
   ┌──────▼──────┐    ┌────────▼──────┐
   │ Spoke 1     │    │ Spoke 2       │
   │ 10.1.0.0/16 │    │ 10.2.0.0/16   │
   │ (Produção)  │    │ (Dev/Test)    │
   └─────────────┘    └───────────────┘
```

- **NSG (Network Security Group):** Regras de entrada/saída por porta, protocolo e IP
- **Azure Firewall:** Firewall gerenciado com regras de aplicação, rede e DNAT
- **Private Endpoints:** Conectividade privada para serviços PaaS (sem tráfego pela internet)

</details>

---

## 💾 Armazenamento Azure

<div align="center">

| Tipo de Redundância | Cópias | Escopo | Uso Recomendado |
|--------------------|--------|--------|-----------------|
| **LRS** | 3 | Datacenter único | Dev/Test, baixo custo |
| **ZRS** | 3 | Zonas na região | Alta disponibilidade zonal |
| **GRS** | 6 | 2 regiões | DR inter-regional |
| **GZRS** | 6 | Zonas + 2 regiões | Máxima resiliência |

</div>

**Camadas de Acesso (Blobs):**
- 🔥 **Hot** — Dados acessados frequentemente (maior custo de armazenamento, menor de acesso)
- ❄️ **Cool** — Dados acessados com pouca frequência (~30 dias)
- 🧊 **Archive** — Dados raramente acessados (custo mínimo, latência de horas para rehidratação)

---

## 🗄️ Bancos de Dados e Alta Disponibilidade

<details>
<summary>📊 Estratégias de Backup, Replicação e Monitoramento</summary>

| Recurso | Funcionalidade |
|---------|----------------|
| **Backup automático** | RPO configurável, retenção até 35 dias (PITR) |
| **Geo-replicação** | Réplica de leitura em região secundária (Azure SQL) |
| **Always On AG** | Grupos de disponibilidade para SQL Server em VMs |
| **Azure Monitor** | Métricas, logs, alertas e dashboards centralizados |
| **Log Analytics** | Consultas KQL, workbooks e análise de telemetria |
| **Azure Advisor** | Recomendações de performance, segurança e custo |

</details>

---

## 🛡️ Governança Azure

<div align="center">

[![Blueprints](https://img.shields.io/badge/Azure%20Blueprints-0078D4?style=flat-square)]()
[![Policies](https://img.shields.io/badge/Azure%20Policy-2E75B6?style=flat-square)]()
[![Arc](https://img.shields.io/badge/Azure%20Arc-16265F?style=flat-square)]()

</div>

- **Azure Blueprints:** Implantação repetível e auditável de ambientes completos (políticas + RBAC + ARM)
- **Azure Policy:** Definição e aplicação de regras de conformidade em escala (deny, audit, deployIfNotExists)
- **Resource Locks:** ReadOnly e Delete — proteção contra alterações acidentais em recursos críticos
- **Azure Arc:** Estende gerenciamento Azure para servidores on-premises, outras nuvens e edge

---

## 🧰 Stack Tecnológico

<div align="center">

[![My Skills](https://skillicons.dev/icons?i=azure,terraform,powershell,git&theme=dark)](https://skillicons.dev)

</div>

---

## 🚀 Como Utilizar

```bash
# 1. Clone o repositório
git clone https://github.com/fassir/resumo-do-lab.git
cd resumo-do-lab

# 2. Configure as credenciais Azure
az login
az account set --subscription "<ID_DA_ASSINATURA>"

# 3. Inicialize o Terraform
cd terraform
terraform init
terraform plan -out=tfplan
terraform apply tfplan

# 4. Execute o script de deploy
cd ../scripts
./deploy.ps1 -ResourceGroupName "rg-lab" -Location "brazilsouth"
```

---

## 📚 Referências

- [Documentação Microsoft Azure](https://docs.microsoft.com/azure)
- [Terraform Provider AzureRM](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Azure DevOps YAML Schema](https://docs.microsoft.com/azure/devops/pipelines/yaml-schema)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture)

---

## 👤 Autor

<div align="center">

**Fabio Piassi**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/fabio-piassi)
[![GitHub](https://img.shields.io/badge/GitHub-171515?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fassir)

</div>

---

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:16265F,50:2E75B6,100:1F9BD4&height=120&section=footer" width="100%"/>
