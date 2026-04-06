# az-ad-vm

Terraform module that provisions a Windows Server 2022 Active Directory Domain Controller on Azure. Infrastructure is fully automated via a Custom Script Extension — no manual configuration post-deploy.

## Stack

- **IaC:** Terraform + AzureRM provider
- **Compute:** Windows Server 2022 (Azure VM)
- **Identity:** AD DS, promoted to new forest via Custom Script Extension
- **Networking:** VNet, Subnet, NSG, Static Public IP

## Usage

```hcl
yourname       = "yourname"
location       = "eastus"
admin_password = "..."
dsrm_password  = "..."
domain_name    = "corp.example.com"
domain_netbios = "CORP"
```

```bash
terraform init && terraform plan && terraform apply
```

> `terraform.tfvars` is gitignored — credentials are never committed.

## Outputs

```bash
terraform output public_ip   # Use for RDP access
```

RDP credentials: `CORP\adadmin` — domain auth is active immediately post-deploy.

## Validation

```powershell
Get-Service NTDS | Select-Object Name, Status
Get-ADDomain
Get-ADDomainController -Filter *
Resolve-DnsName corp.example.com
```

## Teardown

```bash
terraform destroy
```
