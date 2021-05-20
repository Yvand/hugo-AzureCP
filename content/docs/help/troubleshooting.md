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

## Check the SharePoint logs

AzureCP records all its activity in SharePoint logs, including the performance, queries and number of results returned for each LDAP server.

Get AzureCP logging level:

```powershell
Get-SPLogLevel| ?{$_.Area -like "AzureCP"}
```

Set AzureCP logging level:

```powershell
"AzureCP:*"| Set-SPLogLevel -TraceSeverity Verbose
```

Merge AzureCP logs from all servers from the past 10 minutes:

```powershell
Merge-SPLogFile -Path "C:\Data\LDAPCP_logging.log" -Overwrite -Area "AzureCP" -StartTime (Get-Date).AddMinutes(-10)
```

You can use [ULSViewer](https://www.microsoft.com/en-us/download/details.aspx?id=44020) to apply this filter and monitor the logs in real time.

## Test Microsoft Graph queries in Postman

You can import the collection below in [Postman](https://www.postman.com/) to replay the typical queries issued by AzureCP:

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/7f2fca601fa9be1d8bb8)

## Test connectivity with Azure AD

AzureCP may fail to connect to Azure AD for various reasons. The PowerShell script below connects to the typical Azure endpoints and may be run on the SharePoint servers to test the connectivity:

```powershell
Invoke-WebRequest -Uri https://login.windows.net -UseBasicParsing
Invoke-WebRequest -Uri https://login.microsoftonline.com -UseBasicParsing
Invoke-WebRequest -Uri https://graph.microsoft.com -UseBasicParsing
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

## Inspect the actual traffic between AzureCP and Azure AD

You can redirect the traffic between AzureCP and Azure AD through Fiddler to inspect it.
Once Fiddler was installed locally and its root certificate trusted, you can do it per web application by updating the web.config:

```xml
<system.net>
    <defaultProxy useDefaultCredentials="True">
        <proxy proxyaddress="http://localhost:8888" bypassonlocal="False" />
    </defaultProxy>
</system.net>
```
