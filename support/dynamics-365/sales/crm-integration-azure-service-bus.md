---
title: CRM integration with Azure Service Bus
description: This article applies to Microsoft Dynamics CRM Government Cloud organizations that have implemented integration with the Microsoft Azure Service Bus.
ms.reviewer: 
ms.topic: article
ms.date: 
---
# Microsoft Dynamics CRM Government Cloud organization integration with the Microsoft Azure Service Bus

This article applies to Microsoft Dynamics CRM Government Cloud organizations that have implemented integration with the Microsoft Azure Service Bus.

_Applies to:_ &nbsp; Microsoft Dynamics CRM Online  
_Original KB number:_ &nbsp; 3200643

## Summary

Recent security enhancements require the Microsoft Dynamics CRM Online service to use a new certificate to authenticate against the Microsoft Azure service. Use the steps in this article to change the configuration in your Microsoft Azure namespace. These changes are necessary, and will allow the messages sent from the Microsoft Dynamics CRM Online service to the Microsoft Azure service endpoint to be authenticated with both the current certificate and the newer certificate that will be available soon.

> [!NOTE]
> This information also applies to the Dynamics Marketing/Dynamics CRM connector integration.

To ensure minimal impact, this configuration change should be made before 1AM UTC, Tuesday, November 1, 2016 globally.

> [!NOTE]
>
> - Don't remove the old certificate until after 1AM UTC, Friday, November 4th 2016, as the new one isn't valid until this date. However, both the new and old certificates can exist simultaneously without issues.
> - If your organization is using Dynamics CRM version 8.1 or later, then we highly suggest configuring your service endpoints to use SAS authentication instead ACS. For more information, see [Walkthrough: Configure Microsoft Azure (SAS) for integration with Dynamics 365](/previous-versions/dynamicscrm-2016/developers-guide/mt697580(v=crm.8)).
>
> If these changes aren't made, any integrations to Microsoft Dynamics CRM Online that use the Microsoft Azure Service bus will stop working. Also, if the PluginRegistration tool is used to verify authentication, an error message may occur similar to the following example:

> "The token provider was unable to provide a security token. The remote server returned an error: (401) Unauthorized ".

## More information

When the procedures in this article have been completed, ACS access control will be configured to allow Microsoft Dynamics CRM Online to continue to send messages with the new certificates.

First, retrieve the list of service endpoints. The steps in this article will need to be done for each of the service endpoints. To find the service endpoints in Microsoft Dynamics CRM, navigate to **Settings**, select **Customizations**, select **Customize the System**, and select **Service Endpoints**.

:::image type="content" source="media/crm-integration-azure-service-bus/retrieve-service-endpoints.jpg" alt-text="Retrieve the list of service endpoints.":::

> [!NOTE]
> If the service endpoint connection mode is **Federated**, the same steps will need to be repeated in the following instructions for `https://.accesscontrol.windows.net/v2/mgmt/web` or `https://.accesscontrol.usgovcloudapi.net/v2/mgmt/web`.

To configure access control for a service namespace:

1. In a web browser, go to `https://%3cservicenamespace%3e-sb.accesscontrol.windows.net/v2/mgmt/web` or `https://%3cservicenamespace%3e-sb.accesscontrol.usgovcloudapi.net/v2/mgmt/web`.

    If you don't have access, contact the solution developer to follow the steps.

    :::image type="content" source="media/crm-integration-azure-service-bus/access-control-service.jpg" alt-text="Contact the solution developer.":::

2. Under **Service Settings**, select **Service Identities**.

3. Select your Microsoft Dynamics CRM Online service identity to continue to the Edit Service Identity page.

    > [!NOTE]
    > If your organization URL contains `crm9.dynamics.com`, [click here](https://go.microsoft.com/fwlink/?linkid=832559 ) to download the public certificate and save it to your disk. Also, select the check box next to `crm9.dynamics.com`.

    :::image type="content" source="media/crm-integration-azure-service-bus/select-check-box.jpg" alt-text="Select the check box.":::

4. Select **Add**.

    :::image type="content" source="media/crm-integration-azure-service-bus/edit-service-identity.jpg" alt-text="Select add.":::

5. Under **Type**, choose **X509**, and then select **Add**. In the **Add Credential** screen (shown below), browse to the public certificate you previously saved to disk, and select **Save**.

    :::image type="content" source="media/crm-integration-azure-service-bus/add-credential.jpg" alt-text="Save the certificate you previously saved to disk.":::

6. You should now see the current (soon to expire) and new certificates in the Credentials list.

    :::image type="content" source="media/crm-integration-azure-service-bus/new-certificates.jpg" alt-text="The new certificates are added successfully.":::
