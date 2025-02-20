---
title: Connect Azure Communications Gateway to Microsoft Teams Direct Routing
description:  After deploying Azure Communications Gateway, you can configure it to connect to the Microsoft Phone System for Microsoft Teams Direct Routing.
author: rcdun
ms.author: rdunstan
ms.service: communications-gateway
ms.topic: integration
ms.date: 01/08/2024
ms.custom:
    - template-how-to-pattern
---

# Connect Azure Communications Gateway to Microsoft Teams Direct Routing

After you deploy Azure Communications Gateway and connect it to your core network, you need to connect it to Microsoft Phone System.

This article describes how to start connecting Azure Communications Gateway to Microsoft Teams Direct Routing. After you finish the steps in this article, you can set up test users for test calls and prepare for live traffic.

## Prerequisites

You must [deploy Azure Communications Gateway](deploy.md).

Your organization must [integrate with Azure Communications Gateway's Provisioning API](integrate-with-provisioning-api.md). If you didn't configure the Provisioning API in the Azure portal as part of deploying, you also need to know:
- The IP addresses or address ranges (in CIDR format) in your network that should be allowed to connect to the Provisioning API, as a comma-separated list.
- (Optional) The name of any custom SIP header that Azure Communications Gateway should add to messages entering your network.

You must have **Reader** access to the subscription into which Azure Communications Gateway is deployed.

You must be able to sign in to the Microsoft 365 admin center for your tenant as a Global Administrator.

## Enable Microsoft Teams Direct Routing support

> [!NOTE]
> If you selected Microsoft Teams Direct Routing when you [deployed Azure Communications Gateway](deploy.md), skip this step and go to [Find your Azure Communication Gateway's domain names](#find-your-azure-communication-gateways-domain-names).

1. Sign in to the [Azure portal](https://azure.microsoft.com/).
1. In the search bar at the top of the page, search for your Communications Gateway resource and select it.
1. In the side menu bar, find **Communications services** and select **Teams Direct Routing** to open a page for the service.
1. On the service's page, select **Teams Direct Routing settings**.
1. Fill in the fields, selecting **Review + create** and **Create**.
1. Select the **Overview** page for your resource.
1. Wait for your resource to be updated. When your resource is ready, the **Provisioning Status** field on the resource overview changes to "Complete." We recommend that you check in periodically to see if the Provisioning Status field is "Complete." This step might take up to two weeks.

## Find your Azure Communication Gateway's domain names

Before starting this step, check that the **Provisioning Status** field for your resource is "Complete".

Microsoft Teams only sends traffic to domains that you confirm that you own. Your Azure Communications Gateway deployment automatically receives an autogenerated fully qualified domain name (FQDN) and regional subdomains of this domain.

1. Sign in to the [Azure portal](https://azure.microsoft.com/).
1. In the search bar at the top of the page, search for your Communications Gateway resource.
1. Select your Communications Gateway resource. Check that you're on the **Overview** of your Azure Communications Gateway resource.
1. Select **Properties**.
1. Find the field named **Domain**. This name is your deployment's _base domain name_.
1. In each **Service Location** section, find the **Hostname** field. This field provides the _per-region domain name_.
    - A production deployment has two service regions and therefore two per-region domain names.
    - A lab deployment has one service region and therefore one per-region domain name.
1. Note down the base domain name and the per-region domain name(s). You'll need these values in the next steps.

## Register the base domain name for Azure Communications Gateway in your tenant

You need to register the base domain for Azure Communications Gateway in your tenant and verify it. Registering and verifying the base domain proves that you control the domain.

> [!TIP]
> If the base domain name is a subdomain of a domain already registered and verified in this tenant:
> - You must register Azure Communications Gateway's base domain name.
> - Microsoft 365 automatically verifies the base domain name.

Follow the instructions [to add a base domain to your tenant](/microsoftteams/direct-routing-sbc-multiple-tenants#add-a-base-domain-to-the-tenant-and-verify-it). Use the base domain name that you found in [Find your Azure Communication Gateway's domain names](#find-your-azure-communication-gateways-domain-names).

If Microsoft 365 prompts you to verify the domain name:

1. Select DNS TXT records as your verification method with **Add a TXT record instead**.
1. Select **Next**, and note the TXT value that Microsoft 365 provides.
1. Provide the TXT value to your onboarding team as part of [Provide additional information to your onboarding team](#provide-additional-information-to-your-onboarding-team).

Don't try to finish verifying the domain name until your onboarding team confirms that DNS records with the TXT value have been set up.

## Provide additional information to your onboarding team

Before your onboarding team can finish onboarding you to the Microsoft Teams Direct Routing environment, you need to provide them with some additional information.

1. Wait for your onboarding team to provide you with a form to collect the additional information. 
1. Complete the form and give it to your onboarding team.

If you don't already have an onboarding team, contact azcog-enablement@microsoft.com, providing your Azure subscription ID and contact details.

## Finish verifying the base domain name in Microsoft 365

> [!NOTE]
> If Microsoft 365 did not prompt you to verify the domain in [Register the base domain name for Azure Communications Gateway in your tenant](#register-the-base-domain-name-for-azure-communications-gateway-in-your-tenant), skip this step.

After your onboarding team confirms that the DNS records have been set up, finish verifying the base domain name in the Microsoft 365 admin center.

1. Sign into the Microsoft 365 admin center as a Global Administrator.
1. Select **Settings** > **Domains**.
1. Select the base domain.
1. On the **Choose your online services** page, clear all options and select **Next**.
1. Select **Finish** on the **Update DNS settings** page.
1. Ensure that the status is **Setup complete**.

## Set up a user or resource account with the base domain and an appropriate license

To activate the base domain in Microsoft 365, you must have at least one user or resource account licensed for Microsoft Teams. For more information, including the licenses you can use, see [Activate the domain name](/microsoftteams/direct-routing-sbc-multiple-tenants#activate-the-domain-name).

## Connect your tenant to Azure Communications Gateway

You must configure your Microsoft 365 tenant with SIP trunks to Azure Communications Gateway. Each trunk connects to one of the per-region domain names that you found in [Find your Azure Communication Gateway's domain names](#find-your-azure-communication-gateways-domain-names).

Use [Connect your Session Border Controller (SBC) to Direct Routing](/microsoftteams/direct-routing-connect-the-sbc) and the following configuration settings to set up the trunks.

- For a production deployment, set up two trunks.
- For a lab deployment, set up one trunk.

| Teams Admin Center setting | PowerShell parameter | Value to use (Admin Center / PowerShell) |
| -------------------------- | -------------------- | ------------ |
| **Add an FQDN for the SBC** | `FQDN` |The regional domain of Azure Communications Gateway |
| **Enabled** | `Enabled` | On / True |
| **SIP signaling port** | `SipSignalingPort` | 5063 |
| **Send SIP options** | `SendSIPOptions` | On / True |
| **Forward call history** | `ForwardCallHistory` | On / True|
| **Forward P-Asserted-identity (PAI) header** | `ForwardPAI` | On / True |
| **Concurrent call capacity** | `MaxConcurrentSessions` | Leave as default |
| **Failover response codes** | `FailoverResponseCodes` |Leave as default|
| **Failover times (seconds)** | `FailoverTimeSeconds` |Leave as default|
| **SBC supports PIDF/LO for emergency calls** | `PidfloSupported` | On / True |
| - | `MediaBypass` |- / False|

## Next step

> [!div class="nextstepaction"]
> [Configure a test customer](configure-test-customer-teams-direct-routing.md)
