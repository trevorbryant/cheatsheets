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

## JSON Usage
Using json output for targed attributes.
```bash
az ad user show --upn-or-object-id first.last@contoso.com --query '{name:displayName, UPN:userPrincipalName}' -o json
{
  "UPN": "first.last@consoto.com",
  "name": "first last"
}
```
