---
title: "Troubleshooting"
description: ""
lead: "This article groups tips & tricks to help you troubleshoot AzureCP if it's not working as expected."
date: 2021-05-17T13:24:28Z
lastmod: 2021-05-17T13:24:28Z
draft: false
images: []
menu: 
  docs:
    parent: "help"
weight: 950
toc: true
---

## Inspect the SharePoint logs

AzureCP records all its activity in the SharePoint logs, under Product / Area "AzureCP". It records a lot of information and can be managed with PowerShell:

```powershell
# Show the AzureCP logging level
Get-SPLogLevel| ?{$_.Area -like "AzureCP"}
# Set AzureCP logging level
"AzureCP:*"| Set-SPLogLevel -TraceSeverity Verbose
# Merge AzureCP logs from all servers from the past 10 minutes
Merge-SPLogFile -Path "C:\Data\AzureCP_logging.log" -Overwrite -Area "AzureCP" -StartTime (Get-Date).AddMinutes(-10)
```

You can use [ULS Viewer](https://www.microsoft.com/en-us/download/details.aspx?id=44020) to inspect the logs.

## Run Microsoft Graph queries in Postman

You can import the collection below in [Postman](https://www.postman.com/) to replay the typical queries executed by AzureCP:

[Open Postman collection for AzureCP](https://app.getpostman.com/run-collection/7f2fca601fa9be1d8bb8)

## Test connectivity with Azure AD

AzureCP may fail to connect to Azure AD for various reasons. The PowerShell script below connects to the typical Azure endpoints and may be run on the SharePoint servers to test the connectivity:

```powershell
Invoke-WebRequest -Uri "https://login.windows.net" -UseBasicParsing
Invoke-WebRequest -Uri "https://login.microsoftonline.com" -UseBasicParsing
Invoke-WebRequest -Uri "https://graph.microsoft.com" -UseBasicParsing
```

## Obtain the access token using PowerShell

This PowerShell script requests an access token to Microsoft Graph, as done by AzureCP:

```powershell
$clientId = "<CLIENTID>"
$clientSecret = "<CLIENTSECRET>"
$tenantName = "<TENANTNAME>.onmicrosoft.com"
$headers = @{ "Content-Type" = "application/x-www-form-urlencoded" }
$body = "grant_type=client_credentials&client_id=$clientId&client_secret=$clientSecret&resource=https%3A//graph.microsoft.com/"
$response = Invoke-RestMethod "https://login.microsoftonline.com/$tenantName/oauth2/token" -Method "POST" -Headers $headers -Body $body
$response | ConvertTo-Json
```

## Inspect the traffic to Azure AD

You can intercept and inspect the traffic between AzureCP and Azure AD using [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic).  
Once Fiddler was installed locally and its root certificate trusted, you can intercept the traffic per web application by updating the web.config:

```xml
<system.net>
    <defaultProxy useDefaultCredentials="True">
        <proxy proxyaddress="http://localhost:8888" bypassonlocal="False" />
    </defaultProxy>
</system.net>
```

{{< alert icon="💡" text="To view the traffic in Fiddler, make sure to set the filter to \"All Processes\" or \"Non-Browsers\" (in the bottom left)." />}}
