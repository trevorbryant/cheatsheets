# Azure AD CLI Cheat Sheet
Useful commands, append `| jq` to view results in pretty colors.

## Install Azure CLI
Install instructions for Debian/Ubuntu.
[Install Azure CLI with apt](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest)

## Authentication
### Login
Login to account.
```bash
az login --allow-no-subscriptions
```
Confirm successful login.
```bash
az account show
```
### Logout
Log out of account.
```bash
az logout
```

## Query Users, Groups, or Group Memberships
### Query all users
```bash
az ad user list
```
### Query user
```bash
az ad user show user.name@contoso.com
```
### Query all groups
```bash
az ad group list
```
### Query group
```bash
az ad group show --group "azure-objectid-identifiers-1a2b3c"
```

## Query Usage
Using json example output for targed attributes.
```bash
az ad user show --upn-or-object-id first.last@contoso.com --query '{name:displayName, UPN:userPrincipalName}' -o json
{
  "UPN": "first.last@consoto.com",
  "name": "first last"
}
```
All user objects
```bash
az ad user list --query '[].{name:displayName, UPN:userPrincipalName}' -o json
{
  "UPN": "first.last@consoto.com",
  "name": "first last",
  "UPN": "first.last@consoto.com",
  "name": "first last", 
}

```

## Install Azure DevOps
Add the `azure-devops` extension
```bash
az extension add --name azure-devops
```

## Configure project
Configure and add project
```bash
az devops configure --defaults organization=https://dev.azure.com/contoso project=ContosoWebApp
```
