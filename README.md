# Azure Active Directory Domain Controller — Terraform Deployment

Deploys a Windows Server 2022 VM on Azure and automatically promotes it to an Active Directory Domain Controller using Terraform and a Custom Script Extension.

## What Gets Deployed

| Resource | Name pattern |
|---|---|
| Resource Group | `rg-ad-<yourname>` |
| Virtual Network | `vnet-ad-<yourname>` |
| Subnet | `snet-ad` (10.0.1.0/24) |
| Public IP (Static) | `pip-ad-<yourname>` |
| NSG (RDP open) | `nsg-ad-<yourname>` |
| Network Interface | `nic-ad-<yourname>` |
| Windows Server 2022 VM | `vm-ad-<yourname>` |
| Custom Script Extension | Installs AD DS + promotes to new forest |

## Prerequisites

- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) installed and authenticated (`az login`)
- [Terraform v1.3+](https://developer.hashicorp.com/terraform/install)
- An active Azure subscription

## Project Structure

```
az-ad-vm/
├── main.tf           # All Azure resources + Custom Script Extension
├── variables.tf      # Input variable definitions
├── outputs.tf        # Public IP, domain name, admin username
└── terraform.tfvars  # Your values (passwords, name, domain) — not committed
```

## Configuration

Create a `terraform.tfvars` file with your values:

```hcl
yourname       = "yourname"           # Used in all resource names — keep it short
location       = "eastus"             # Azure region
admin_password = "YourPassword123!"   # VM local admin password
dsrm_password  = "YourDSRM123!"      # AD recovery password — store this securely
domain_name    = "corp.example.com"   # Fully qualified domain name
domain_netbios = "CORP"              # NetBIOS name (max 15 chars)
```

> **Note:** `terraform.tfvars` is excluded from version control via `.gitignore` as it contains sensitive credentials.

## Deploy

```bash
terraform init
terraform plan
terraform apply
```

Apply takes approximately **8–13 minutes** total:
- VM deployment: ~5–8 min
- AD DS installation + reboot: ~3–5 min additional

## Connect via RDP

```bash
terraform output public_ip
```

Use the IP to open an RDP session. **Wait 5–10 minutes after apply completes** before connecting — the VM reboots automatically after AD DS installs.

| Method | Username |
|---|---|
| Domain prefix (recommended) | `CORP\adadmin` |
| UPN format | `adadmin@corp.example.com` |
| Local account (fallback) | `.\adadmin` |

## Verify AD DS

Run these in PowerShell (as Administrator) on the domain controller:

```powershell
# AD DS service is running
Get-Service NTDS | Select-Object Name, Status

# Domain info
Get-ADDomain

# List domain controllers
Get-ADDomainController -Filter *

# DNS resolving
Resolve-DnsName corp.example.com
```

All four commands should return without errors.

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| RDP password rejected | VM promoted to domain — auth changed | Use `CORP\adadmin` instead of local credentials |
| Extension status: Failed | AD DS install failed mid-run | Check `C:\WindowsAzure\Logs` on VM, re-run `terraform apply` |
| RDP black screen | VM still rebooting after AD DS | Wait 5 min and retry |
| `Get-ADDomain` not found | Module not loaded | Run `Import-Module ActiveDirectory` then retry |

## Teardown

```bash
terraform destroy
```

Removes the resource group and everything inside it — VM, disks, NIC, public IP, NSG, VNet, and subnet.
