---
title: "Configure"
description: ""
lead: "Configure AzureCP to fit your needs"
date: 2021-05-17T14:06:29Z
lastmod: 2021-05-17T14:06:29Z
draft: false
images: []
menu: 
  docs:
    parent: "usage"
weight: 810
toc: true
---

## Prerequisites

To perform the configuration, you first need to [register an application]({{< ref "register-application" >}}) in your Azure Active Directory tenant (to allow AzureCP to query it).

## Configure with administration pages

AzureCP comes with 2 administration pages added in central administration > Security:

- Global configuration: Add / remove LDAP servers and configure various settings.
- Claim types configuration: Define the claim types, and their mapping with LDAP objects.

## Configure with PowerShell

Starting with v12, AzureCP can be configured with PowerShell:

```powershell
Add-Type -AssemblyName "AzureCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [azurecp.AzureCPConfig]::GetConfiguration("AzureCPConfig")

# To view current configuration
$config
$config.ClaimTypes

# Update some settings, e.g. enable augmentation:
$config.EnableAugmentation = $true
$config.Update()

# Reset claim types configuration list to default
$config.ResetClaimTypesList()
$config.Update()

# Reset the whole configuration to default
$config.ResetCurrentConfiguration()
$config.Update()

# Add a new Azure AD tenant
$newAADTenant = New-Object azurecp.AzureTenant
$newAADTenant.Name = "xxx.onMicrosoft.com"
$newAADTenant.ApplicationId = "Application ID"
$newAADTenant.ApplicationSecret = "XXX"
$config.AzureTenants.Add($newAADTenant)
$config.Update()

# Add a new entry to the claim types configuration list
$newCTConfig = New-Object azurecp.ClaimTypeConfig
$newCTConfig.ClaimType = "ClaimTypeValue"
$newCTConfig.EntityType = [azurecp.DirectoryObjectType]::User
$newCTConfig.DirectoryObjectProperty = [azurecp.AzureADObjectProperty]::Department
$config.ClaimTypes.Add($newCTConfig)
$config.Update()

# Remove a claim type from the claim types configuration list
$config.ClaimTypes.Remove("ClaimTypeValue")
$config.Update()
```

AzureCP configuration is stored as a persisted object in the SharePoint configuration database, and can be returned with this SQL command:

```sql
SELECT Id, Name, cast (properties as xml) AS XMLProps FROM Objects WHERE Name = 'AzureCPConfig'
```

## Configure proxy for internet access

AzureCP may be used by each SharePoint process (w3wp, owstimer) on any server, so they all need to be able to connect to Azure AD, which is comprised of various endpoints that depend on [the region](https://docs.microsoft.com/en-us/azure/active-directory/develop/authentication-national-cloud) of your Azure AD tenant.
Windows also needs to be able to validate the certificates.

### Configure the proxy for SharePoint processes

If SharePoint servers connect to internet through a proxy, [additional configuration](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/file-schema/network/defaultproxy-element-network-settings) is required in the configuration files:

```xml
<system.net>
    <defaultProxy useDefaultCredentials="True">
        <proxy proxyaddress="http://proxy.contoso.com:8888" bypassonlocal="True" />
    </defaultProxy>
</system.net>
```

This configuration must be set on the following processes, on all SharePoint servers of the farm:

* SharePoint sites that use AzureCP
* SharePoine central administration site
* SharePoint STS located in 16\WebServices\SecurityToken
* SharePoint Web Services root site
* SharePoint Timer service (if necessary, create file owstimer.exe.config in 16\BIN)

### Configure the proxy for Windows processes

Connection to Azure AD is secured, and Windows validates all certificates in the chain.  
If Windows cannot validate them, the usual symptom is a hang during 1 minute upon sign-in, and errors are recorded in CAPI2 event log.  
Certificate validation is performed by lsass.exe, which uses the proxy configuration set with netsh.exe:

```text
# To see proxy configuration
netsh winhttp show proxy
# To see proxy configuration
netsh winhttp set proxy proxy-server="http=myproxy;https=sproxy:88" bypass-list="*.foo.com"
# To remove proxy configuration
netsh winhttp reset proxy
```
