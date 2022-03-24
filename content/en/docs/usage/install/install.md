---
title: "Install"
description: "Install AzureCP"
lead: "Install AzureCP in your SharePoint farm"
date: 2021-05-20T10:45:52Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
menu: 
  docs:
    parent: "usage"
weight: 800
toc: true
---

## Important

SharePoint (especially 2019) has unaddressed reliability issues when deploying farm solutions on farms with multiple servers.  
The more servers the farm has, the bigger the risk that deployment fails. To mitigate this, cmdlet `Install-SPSolution` can be run with `-Local` on each server, but it requires more operations.  
This page will guide you through the steps to install AzureCP in a safe and reliable way.

## Deploy the solution

{{< alert icon="💡" text="Always start a new PowerShell process to ensure using up to date persisted objects and avoid nasty errors." />}}

Execute the following steps:

- Download [the latest version](https://github.com/Yvand/AzureCP/releases/latest) of AzureCP.wsp.
- Install and deploy the solution, using either the __simple__ or the __safe__ method:
  - __Simple__ method: Recommended for single-server farms only:

  ```powershell
  # Run this script on the server running central administration, in a new PowerShell process
  Add-SPSolution -LiteralPath "C:\Data\AzureCP.wsp"
  # Wait for some time (until solution is actually added)
  # Then run Install-SPSolution (without -Local) to deploy solution globally (on all servers that run service "Microsoft SharePoint Foundation Web Application"):
  Install-SPSolution -Identity "AzureCP.wsp" -GACDeployment
  ```
  
  - __Safe__ method: Recommended for production environments with multiple servers:

  {{< alert icon="💡" text="Run this script on ALL SharePoint servers running the service \"Microsoft SharePoint Foundation Web Application\" and/or the central administration, sequentially (not in parallel), starting with the server running the central administration." />}}

  ```powershell
  <#
  .SYNOPSIS
      Deploy "AzureCP.wsp" in a reliable way, to address reliability issues that may occur when deploying solutions in SharePoint (especially 2019) (and especially if there are many servers):
  .DESCRIPTION
      Run this script on ALL SharePoint servers running the service "Microsoft SharePoint Foundation Web Application" and/or the central administration, sequentially (not in parallel), starting with the server running the central administration.
      The script does not require any modification, you only need to set the variable $packagefullpath with the path to the solution file (used only on the 1st server)
  .LINK
      https://www.AzureCP.com/docs/usage/install/
  #>

  # To use this script, you only need to edit the variable $packagefullpath below
  $claimsprovider = "AzureCP"
  $packagefullpath = "C:\Data\$claimsprovider.wsp"
  $packageName = "$claimsprovider.wsp"

  # Perform checks on the local server to detect and prevent potential problems
  # Check 1: Install-SPSolution will fail if claims provider is already installed on the current server
  if ($null -ne (Get-SPClaimProvider -Identity $claimsprovider -ErrorAction SilentlyContinue)) {
      Write-Error "Cannot continue because current server already has claims provider $claimsprovider, which will cause an error when running Install-SPSolution."
      throw ("Cannot continue because current server already has claims provider $claimsprovider, which will cause an error when running Install-SPSolution.")
      Get-SPClaimProvider| ?{$_.DisplayName -like $claimsprovider}| Remove-SPClaimProvider
  }

  # Check 2: Install-SPSolution will fail if any feature in the WSP solution is already installed on the current server
  if ($null -ne (Get-SPFeature| ?{$_.DisplayName -like "$claimsprovider*"})) {
      Write-Error "Cannot continue because current server already has features of $claimsprovider, Visit https://www.AzureCP.com/docs/help/fix-setup-issues/ to fix this."
      throw ("Cannot continue because current server already has features of $claimsprovider, Visit https://www.AzureCP.com/docs/help/fix-setup-issues/ to fix this.")
  }

  Write-Host "All checks passed on this server, continuing..."
  # Add the solution if it's not already present in the farm (only the 1st server will actually do this)
  if ($null -eq (Get-SPSolution -Identity $packageName -ErrorAction SilentlyContinue)) {
      Write-Host "Adding solution $packageName to the farm..."
      Add-SPSolution -LiteralPath $packagefullpath
  }

  $count = 0
  while (($count -lt 20) -and ($null -eq $solution))
  {
      Write-Host "Waiting for the solution $packageName to be available..."
      Start-Sleep -Seconds 5
      $solution = Get-SPSolution -Identity $packageName
      $count++
  }

  if ($null -eq $solution) {
      Write-Error "Solution $packageName could not be found in the farm."
      throw ("Solution $packageName could not be found in the farm.")
  }

  # Always wait at least 5 seconds to avoid that Install-SPSolution does not actually trigger deployment
  Start-Sleep -Seconds 5
  Write-Host "Deploying solution $packageName on the local server..."
  # Set -local in Install-SPSolution to deploy the bits on this server only and prevent reliability issues in SharePoint
  Install-SPSolution -Identity $packageName -GACDeployment -Local
  ```

- Visit central administration > System Settings > Manage farm solutions: Confirm the solution is "Globally deployed".

{{< alert icon="💡" text="If you did not run `Install-SPSolution -Local` on every SharePoint server running the service \"Microsoft SharePoint Foundation Web Application\" and/or the central administration, the solution will NOT be \"Globally deployed\" and SharePoint will NOT activate AzureCP features." />}}

## Finalize the installation

This step is **very important** and applies to **all** SharePoint servers which do **NOT run the service "Microsoft SharePoint Foundation Web Application" and/or the central administration**.  
{{< alert icon="💡" text="This step must be executed during both install (`Install-SPSolution`) and update (`Update-SPSolution`) scenarios." />}}
For each of those servers, complete the steps below to manually add/update AzureCP bits (azurecp.dll and its dependent assemblies) in the GAC:

- Download the package ['AzureCP-XXXX-dependencies.zip'](https://github.com/Yvand/AzureCP/releases) corresponding to the version you downloaded (expand the "Assets" to find it).
- Use [7-zip](https://www.7-zip.org/) to extract the DLLs in a local folder.
- Run the script below to add the assemblies in the GAC:

  ```powershell
  <#
  .SYNOPSIS
      Check if assemblies in the directory specified are present in the GAC, and add them if not.
  .DESCRIPTION
      This script needs to be executed each time you install/update AzureCP, on all SharePoint servers that do not run SharePoint service “Microsoft SharePoint Foundation Web Application”.
      Set $assemblies to the path where 'AzureCP-XXXX-dependencies.zip' was unzipped
  .LINK
      https://azurecp.yvand.net/docs/usage/install/
  #>

  $assemblies = Get-ChildItem -Path "C:\AzureCP-XXXX-dependencies-unzipped\*.dll"

  [System.Reflection.Assembly]::Load("System.EnterpriseServices, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
  $publish = New-Object System.EnterpriseServices.Internal.Publish
  foreach ($assembly in $assemblies)
  {
      $filePath = $assembly.FullName
      $assemblyFullName = [System.Reflection.AssemblyName]::GetAssemblyName($filePath).FullName

      try
      {
          $assemblyInGAC = [System.Reflection.Assembly]::Load($assemblyFullName)
          Write-Host "Assembly $assemblyFullName is already present in the GAC"

          if ($assemblyInGAC.FullName -match "AzureCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636")
          {
              # AzureCP must always be overwritten because each version has the same full name
              $publish.GacRemove($assemblyInGAC)
              $publish.GacInstall($filePath)
              Write-Host "Assembly $assemblyFullName sucessfully re-added to the GAC" -ForegroundColor Green
          }
      }
      catch [System.IO.FileNotFoundException] 
      {
          # Assembly.Load throws a FileNotFoundException if assembly is not found in the GAC: https://docs.microsoft.com/en-us/dotnet/api/system.io.filenotfoundexception?view=netframework-4.8
          Write-Host "Adding assembly $assemblyFullName to the GAC"
          $publish.GacInstall($filePath)
          Write-Host "Assembly $assemblyFullName sucessfully added to the GAC" -ForegroundColor Green
      }
  }
  ```

- Restart the IIS service and the SharePoint timer service (SPTimerV4):  
`Restart-Service W3SVC; Restart-Service SPTimerV4`

## Enable the claims provider

To be enabled, AzureCP must be associated with the SPTrustedLoginProvider which stores the configuration of the trust with the trusted STS:

```powershell
$trust = Get-SPTrustedIdentityTokenIssuer "SPTRUST NAME"
$trust.ClaimProviderName = "AzureCP"
$trust.Update()
```

You can now [configure AzureCP]({{< ref "configuration" >}}).
