---
title: "Introduction"
description: ""
lead: ""
date: 2021-05-26T08:14:13Z
lastmod: 2021-08-04T11:15:29Z
draft: false
images: []
menu: 
  docs:
    parent: "overview"
weight: 100
toc: true
---

## Use case

AzureCP is useful when SharePoint is federated with Azure Active Directory using [WS-Federation](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/sharepoint-on-premises-tutorial) or [OpenID Connect](https://docs.microsoft.com/en-us/sharepoint/security-for-sharepoint-server/oidc-1-0-authentication).  
It runs inside SharePoint and queries your Azure Active Directory tenant(s) to find users and groups:

![people-picker-AzureCP](/images/people-picker-AzureCP.png)

To allow AzureCP to query your tenant, you need to [register a specific application]({{< ref "/docs/usage/register-application" >}}) in your Azure Active Directory tenant and grant it some permissions.

## Compatibility

AzureCP is fully compatible with all the supported versions of SharePoint Server: SharePoint Subscription, SharePoint 2019, SharePoint 2016 and SharePoint 2013.

## Features

- Improve the search in the people picker
- Query multiple Azure Active Directory tenants in parallel.
- Get group membership of Azure AD users (augmentation).
- Populate the metadata (e.g. email, display name) of the entities.
- Easy to configure through PowerShell or administration pages.
- No dependency on any SharePoint service application.

## Customization

AzureCP is highly customizable to adapt to your requirements:

- Connect to your Azure Active Directory tenant using either a client secret or a certificate.
- Customize the display of the results in the people picker.
- Customize the claim types and their mapping with Azure AD objects.
- Enable/disable augmentation.
- Enable/disable connection to Azure AD, to keep AzureCP running with limited functionality if connectivity with Azure AD is lost.
- Developers can deeply [customize AzureCP]({{< ref "for-developers" >}}) to meet specific needs.
