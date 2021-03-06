﻿---
title: Map an existing custom DNS name to Azure Web Apps | Microsoft Docs 
description: Learn how to add an existing custom DNS domain name (vanity domain) to a web app, mobile app backend, or API app in Azure App Service.
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: ''

ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 06/23/2017
ms.author: cephalin
ms.custom: mvc
---
# Map an existing custom DNS name to Azure Web Apps

[Azure Web Apps](app-service-web-overview.md) provides a highly scalable, self-patching web hosting service. This tutorial shows you how to map an existing custom DNS name to Azure Web Apps.

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Map a subdomain (for example, `www.contoso.com`) by using a CNAME record
> * Map a root domain (for example, `contoso.com`) by using an A record
> * Map a wildcard domain (for example, `*.contoso.com`) by using a CNAME record
> * Automate domain mapping with scripts

You can use either a **CNAME record** or an **A record** to map a custom DNS name to App Service. 

> [!NOTE]
> We recommend that you use a CNAME for all custom DNS names except a root domain (for example, `contoso.com`).

To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](app-service-custom-domain-name-migrate.md).

## Prerequisites

To complete this tutorial:

* [Create an App Service app](/azure/app-service/), or use an app that you created for another tutorial.
* Purchase a domain name and make sure you have access to the DNS registry for your domain provider (such as GoDaddy).

  For example, to add DNS entries for `contoso.com` and `www.contoso.com`, you must be able to configure the DNS settings for the `contoso.com` root domain.

  > [!NOTE]
  > If you don't have an existing domain name, consider [purchasing a domain using the Azure portal](custom-dns-web-site-buydomains-web-app.md). 

## Prepare the app

To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**). In this step, you make sure that the App Service app is in the supported pricing tier.

### Sign in to Azure

Open the [Azure portal](https://portal.azure.com) and sign in with your Azure account.

### Navigate to the app in the Azure portal

From the left menu, select **App Services**, and then select the name of the app.

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/select-app.png)

You see the management page of the App Service app.  

<a name="checkpricing"></a>

### Check the pricing tier

In the left navigation of the app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.

![Scale-up menu](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

The app's current tier is highlighted by a blue border. Check to make sure that the app is not in the **Free** tier. Custom DNS is not supported in the **Free** tier. 

![Check pricing tier](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

If the App Service plan is not **Free**, close the **Choose your pricing tier** page and skip to [Map a CNAME record](#cname).

<a name="scaleup"></a>

### Scale up the App Service plan

Select any of the non-free tiers (**Shared**, **Basic**, **Standard**, or **Premium**). 

Click **Select**.

![Check pricing tier](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

When you see the following notification, the scale operation is complete.

![Scale operation confirmation](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

<a name="cname"></a>

## Map a CNAME record

In the tutorial example, you add a CNAME record for the `www` subdomain (for example, `www.contoso.com`).

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### Create the CNAME record

Add a CNAME record to map a subdomain to the app's default hostname (`<app_name>.azurewebsites.net`).

For the `www.contoso.com` domain example, add a CNAME record that maps the name `www` to `<app_name>.azurewebsites.net`.

After you add the CNAME, the DNS records page looks like the following example:

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/cname-record.png)

### Enable the CNAME record mapping in Azure

In the left navigation of the app page in the Azure portal, select **Custom domains**. 

![Custom domain menu](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

In the **Custom domains** page of the app, add the fully qualified custom DNS name (`www.contoso.com`) to the list.

Select the **+** icon next to **Add hostname**.

![Add host name](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

Type the fully qualified domain name that you added a CNAME record for, such as `www.contoso.com`. 

Select **Validate**.

The **Add hostname** button is activated. 

Make sure that **Hostname record type** is set to **CNAME (www.example.com or any subdomain)**.

Select **Add hostname**.

![Add DNS name to the app](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

It might take some time for the new hostname to be reflected in the app's **Custom domains** page. Try refreshing the browser to update the data.

![CNAME record added](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

If you missed a step or made a typo somewhere earlier, you see a verification error at the bottom of the page.

![Verification error](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

<a name="a"></a>

## Map an A record

In the tutorial example, you add an A record for the root domain (for example, `contoso.com`). 

<a name="info"></a>

### Copy the app's IP address

To map an A record, you need the app's external IP address. You can find this IP address in the app's **Custom domains** page in the Azure portal.

In the left navigation of the app page in the Azure portal, select **Custom domains**. 

![Custom domain menu](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

In the **Custom domains** page, copy the app's IP address.

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### Create the A record

To map an A record to an app, App Service requires **two** DNS records:

- An **A** record to map to the app's IP address.
- A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`. App Service uses this record only at configuration time, to verify that you own the custom domain. After your custom domain is validated and configured in App Service, you can delete this TXT record. 

For the `contoso.com` domain example, create the A and TXT records according to the following table (`@` typically represents the root domain). 

| Record type | Host | Value |
| - | - | - |
| A | `@` | IP address from [Copy the app's IP address](#info) |
| TXT | `@` | `<app_name>.azurewebsites.net` |

When the records are added, the DNS records page looks like the following example:

![DNS records page](./media/app-service-web-tutorial-custom-domain/a-record.png)

<a name="enable-a"></a>

### Enable the A record mapping in the app

Back in the app's **Custom domains** page in the Azure portal, add the fully qualified custom DNS name (for example, `contoso.com`) to the list.

Select the **+** icon next to **Add hostname**.

![Add host name](./media/app-service-web-tutorial-custom-domain/add-host-name.png)

Type the fully qualified domain name that you configured the A record for, such as `contoso.com`.

Select **Validate**.

The **Add hostname** button is activated. 

Make sure that **Hostname record type** is set to **A record (example.com)**.

Select **Add hostname**.

![Add DNS name to the app](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

It might take some time for the new hostname to be reflected in the app's **Custom domains** page. Try refreshing the browser to update the data.

![A record added](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

If you missed a step or made a typo somewhere earlier, you see a verification error at the bottom of the page.

![Verification error](./media/app-service-web-tutorial-custom-domain/verification-error.png)

<a name="wildcard"></a>

## Map a wildcard domain

In the tutorial example, you map a [wildcard DNS name](https://en.wikipedia.org/wiki/Wildcard_DNS_record) (for example, `*.contoso.com`) to the App Service app by adding a CNAME record. 

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### Create the CNAME record

Add a CNAME record to map a wildcard name to the app's default hostname (`<app_name>.azurewebsites.net`).

For the `*.contoso.com` domain example, the CNAME record will map the name `*` to `<app_name>.azurewebsites.net`.

When the CNAME is added, the DNS records page looks like the following example:

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)

### Enable the CNAME record mapping in the app

You can now add any subdomain that matches the wildcard name to the app (for example, `sub1.contoso.com` and `sub2.contoso.com` match `*.contoso.com`). 

In the left navigation of the app page in the Azure portal, select **Custom domains**. 

![Custom domain menu](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

Select the **+** icon next to **Add hostname**.

![Add host name](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

Type a fully qualified domain name that matches the wildcard domain (for example, `sub1.contoso.com`), and then select **Validate**.

The **Add hostname** button is activated. 

Make sure that **Hostname record type** is set to **CNAME record (www.example.com or any subdomain)**.

Select **Add hostname**.

![Add DNS name to the app](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

It might take some time for the new hostname to be reflected in the app's **Custom domains** page. Try refreshing the browser to update the data.

Select the **+** icon again to add another hostname that matches the wildcard domain. For example, add `sub2.contoso.com`.

![CNAME record added](./media/app-service-web-tutorial-custom-domain/cname-record-added-wildcard2.png)

## Test in browser

Browse to the DNS name(s) that you configured earlier (for example, `contoso.com`,  `www.contoso.com`, `sub1.contoso.com`, and `sub2.contoso.com`).

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

## Automate with scripts

You can automate management of custom domains with scripts, using the [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell](/powershell/azure/overview). 

### Azure CLI 

The following command adds a configured custom DNS name to an App Service app. 

```bash 
az appservice web config hostname add \
    --webapp <app_name> \
    --resource-group <resource_group_name> \ 
    --name <fully_qualified_domain_name> 
``` 

For more information, see [Map a custom domain to a web app](scripts/app-service-cli-configure-custom-domain.md). 

### Azure PowerShell 

The following command adds a configured custom DNS name to an App Service app. 

```PowerShell  
Set-AzureRmWebApp `
    -Name <app_name> `
    -ResourceGroupName <resource_group_name> ` 
    -HostNames @("<fully_qualified_domain_name>","<app_name>.azurewebsites.net") 
```

For more information, see [Assign a custom domain to a web app](scripts/app-service-powershell-configure-custom-domain.md).

## Next steps

In this tutorial, you learned how to:

> [!div class="checklist"]
> * Map a subdomain by using a CNAME record
> * Map a root domain by using an A record
> * Map a wildcard domain by using a CNAME record
> * Automate domain mapping with scripts

Advance to the next tutorial to learn how to bind a custom SSL certificate to a web app.

> [!div class="nextstepaction"]
> [Bind an existing custom SSL certificate to Azure Web Apps](app-service-web-tutorial-custom-ssl.md)
