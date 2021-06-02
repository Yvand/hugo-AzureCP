---
title: "Limitations"
description: ""
lead: "AzureCP has limitations which SharePoint administrators should be aware of before installing it."
date: 2021-06-01T12:35:32Z
lastmod: 2021-06-01T12:35:32Z
draft: false
images: []
menu: 
  docs:
    parent: "overview"
weight: 150
toc: true
---


## When AzureCP cannot be used

- SharePoint servers have no network access to Azure Active Directory.
- Cmdlet `New-SPTrustedIdentityTokenIssuer` was run with the switch `-UseDefaultConfiguration`.
- It is already associated with a trust, and you want to associate it with a new trust.
