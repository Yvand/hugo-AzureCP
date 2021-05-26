---
title: "Introduction"
description: ""
lead: ""
date: 2021-05-26T08:14:13Z
lastmod: 2021-05-26T08:14:13Z
draft: false
images: []
menu: 
  docs:
    parent: "overview"
weight: 100
toc: true
---

## Use case

AzureCP is useful when SharePoint 2019 / 2016 / 2013 is [federated with Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/sharepoint-on-premises-tutorial).  
It runs inside SharePoint and uses [Microsoft Graph](https://developer.microsoft.com/en-us/graph/) to connect to your Azure Active Directory tenant(s) and return users and groups to SharePoint in various scenarios, such as the people picker.

![Example image](/images/people-picker-AzureCP.png)

## Features

- Query multiple Azure Active Directory tenants in parallel.
- Easy to configure through dedicated pages in central administration, or using PowerShell.
- Return group membership of Azure AD users (augmentation).
- Populate the metadata (e.g. email, display name) of entities.
- No dependency on any SharePoint service application.

## Customization

AzureCP is highly customizable to adapt to your requirements:

- Connect to your Azure Active Directory tenant using either a client secret or a certificate.
- Customize the display of the results in the people picker.
- Customize the claim types and their mapping with Azure AD objects.
- Enable/disable augmentation.
- Enable/disable connection to Azure AD, to keep AzureCP running with limited functionality if connectivity with Azure AD is lost.
- Developers can deeply [customize AzureCP]({{< ref "for-developers" >}}) to meet specific needs.
