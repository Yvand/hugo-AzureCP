---
title: "Fix Setup Issues"
description: ""
lead: "Sometimes, the install/uninstall/update of AzureCP fails. This article will guide through the steps to clean this."
date: 2021-05-17T13:24:40Z
lastmod: 2021-05-17T13:24:40Z
draft: false
images: []
menu: 
  docs:
    parent: "help"
weight: 960
toc: true
---

This procedure will:

- Clean artifacts that were not correctly removed by SharePoint.
- Fully and properly uninstall the solution.

After it is commpleted, AzureCP can be safely re-installed as if it was done for the 1st time.

## Remove AzureCP claims provider

{{< alert icon="💡" text="Always start a new PowerShell process to ensure using up to date persisted objects and avoid nasty errors.<br>Execute all the operations below on the server running the central administration." >}}

This commands removes the SPClaimProvider object from the SharePoint farm:

```powershell
# Remove the claims provider from the current server. There will be no confirmation on the remove
Get-SPClaimProvider| ?{$_.DisplayName -like "AzureCP"}| Remove-SPClaimProvider
```

## Manually create the missing features

This step will recreate the missing folders of features that were physically removed from the filesystem, but not removed from SharePoint configuration database.

1. Identify AzureCP features still installed

```powershell
# Identify all AzureCP features still installed on the current server, that need to be manually uninstalled
Get-SPFeature| ?{$_.DisplayName -like 'AzureCP*'}| fl DisplayName, Scope, Id, RootDirectory
```

Usually, only AzureCP farm feature is listed:

```text
DisplayName   : AzureCP
Scope         : Farm
Id            : d1817470-ca9f-4b0c-83c5-ea61f9b0660d
RootDirectory : C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\Template\Features\AzureCP
```

2. Recreate missing feature folder if needed.

For each feature listed above, check if its "RootDirectory" actually exists in the file system of the current server.  
If it does not exist:

* Create the "RootDirectory". Based on output above, it would be "AzureCP" in folder "16\Template\Features"
* Use [7-zip](http://www.7-zip.org/) to open AzureCP.wsp and extract the feature.xml of the corresponding feature
* Copy the feature.xml into the "RootDirectory"

3. Deactivate and remove the features

```powershell
# Deactivate AzureCP features (it may thrown an error if feature is already deactivated)
Get-SPFeature| ?{$_.DisplayName -like 'AzureCP*'}| Disable-SPFeature -Confirm:$false
# Uninstall AzureCP features on the current server
Get-SPFeature| ?{$_.DisplayName -like 'AzureCP*'}| Uninstall-SPFeature -Confirm:$false
```

4. Once all features are uninstalled, delete the "RootDirectory" folder(s) that was created earlier.

## Delete the AzureCP persisted object

AzureCP stores its configuration is its own persisted object. This stsadm command will delete it if it was not already deleted:

```
stsadm -o deleteconfigurationobject -id 0E9F8FB6-B314-4CCC-866D-DEC0BE76C237
```

## Remove the AzureCP solution

```powershell
# Remove the solution globally (farm wide operation)
Remove-SPSolution "AzureCP.wsp" -Confirm:$false
```

Close PowerShell.  
AzureCP is now properly and completely uninstalled. If desired, you may [re-install it]({{< relref "install" >}}).
