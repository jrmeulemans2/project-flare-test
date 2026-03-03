# Quickstart: Project Flare Static Landing Page

**Feature**: 001-project-flare-landing  
**Architecture**: Azure Front Door → Azure Storage Account (static website), East US, Terraform IaC.

## Prerequisites

- Azure subscription.
- [Terraform](https://www.terraform.io/downloads) installed (e.g., 1.x).
- Azure CLI logged in: `az login`.
- (Optional) Custom domain and DNS access if not using default Front Door hostname.

## 1. Terraform Backend (one-time)

Terraform state should be stored in Azure Storage with locking (constitution). Create a resource group and storage account for state (East US), then a container, e.g.:

```bash
az group create --name rg-terraform-state --location eastus
az storage account create --name <unique-storage-name> --resource-group rg-terraform-state --location eastus --sku Standard_LRS
az storage container create --name tfstate --account-name <unique-storage-name>
```

Configure Terraform backend in `terraform/` (e.g. `backend "azurerm"` with key `project-flare/landing.tfstate`). See Terraform Azure backend docs.

## 2. Provision Infrastructure

From repository root:

```bash
cd terraform
terraform init
terraform plan   # Review: Storage Account + Front Door, East US
terraform apply  # Confirm with yes
```

Note outputs: Front Door hostname (or custom domain), Storage Account name and $web URL (origin).

## 3. Deploy Static Content

Upload the contents of `site/` to the Storage Account’s **$web** container (static website). Options:

- **Azure CLI**: `az storage blob upload-batch -s site -d '$web' --account-name <storage-account-name>`
- **Portal**: Storage Account → Containers → $web → Upload (index.html, css/, assets/).
- **Terraform**: Optional `azurerm_storage_blob` resources or null_resource + local-exec to sync `site/` to $web.

Ensure `index.html` is at the root of $web so the static website default document works.

## 4. Configure Front Door Origin

In Terraform (or Portal): Front Door origin should point at the Storage static website endpoint (e.g. `<account>.z6.web.core.windows.net`). Use HTTP for origin (Front Door terminates HTTPS). If Storage is private, use private link or allow Front Door only; otherwise public read on $web is acceptable for low-cost static.

## 5. Verify

1. Open the Front Door URL in a browser (HTTPS).
2. Confirm Project Flare name and primary message (spec acceptance).
3. Confirm connection is HTTPS (padlock / spec SC-002).
4. Optional: Check that the Storage website URL is not required for normal users (they use Front Door only).

## 6. Tear Down

```bash
cd terraform
terraform destroy
```

Remove state backend resources separately if desired.

## Reference

- [spec.md](./spec.md) – Requirements and acceptance scenarios.
- [plan.md](./plan.md) – Architecture and Constitution Check.
- [research.md](./research.md) – Decisions (Storage, Front Door, East US, Terraform).
