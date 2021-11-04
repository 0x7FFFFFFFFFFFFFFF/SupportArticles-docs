---
title: Support tip for enrolling devices in Intune
description: Brief overview and comparison of the methods that you can use to enroll Windows 10 devices in Microsoft Intune.
ms.date: 09/08/2020
ms.prod-support-area-path: intune
ms.reviewer: sausarka
author: v-tappelgate
ms.author: v-tappelgate
ms.custom: 
- CI 121335
- CSSTroubleshooting
---

# Support Tip &ndash; Comparison of methods for enrolling a Windows device in Intune

This article provides an overview and comparison of the methods that you can use to enroll Windows 10 devices in Microsoft Intune. It also provides an overview of the relevant logs and registry entries.

_Original product version:_&nbsp;Microsoft Intune

## Background

Before you can manage devices in Intune, you have to enroll the devices in the Intune service. You can use Intune to manage both personally owned and corporate-owned devices. You can manually enroll a single device, or automatically enroll multiple devices. The following table compares the two approaches:

| Manual&nbsp;enrollment | Automatic enrollment |
| --- | --- |
|Started by the user. |Started by an automated process, which fetches the user or device credentials. The user doesn't have to sign in. |
|Typically used to enroll personal devices. |Typically used to bulk-enroll corporate devices for an enterprise. |

## Overview of enrollment methods

 :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-enrollment-methods-1.png" alt-text="You can select one of eight methods to enroll devices in Intune":::

You can choose from the following eight methods to enroll a Windows device in Intune:

- **[Manual enrollment in device management](#1).** The user manually enrolls the device in Intune without joining an Azure Active Directory (Azure AD) domain.
- **[Automatic enrollment by using MDM](#2).** The user joins the device to an Azure AD domain, and the device automatically enrolls in Intune.
- **[Automatic enrollment by using GPO](#3).** The user joins the device to an Azure AD domain, and a GPO for the domain automatically enrolls the device in Intune.
- **[Windows Autopilot enrollment](#4).** The device automatically enrolls during the Windows Out-of-the-Box-Experience (OOBE).
- **[Intune and Configuration Manager co-management enrollment](#5).** A device that's already managed by System Center Configuration Manager automatically enrolls in Intune.
- **[Deep link enrollment](#6).** The user selects a link to a web site to start the enrollment.
- **[Company portal app or website enrollment](#7).** The user installs a portal app or goes to a portal website, and selects links to start the enrollment process.
- **[Windows Configuration Designer (WCD) provisioning package enrollment](#8).** A provisioning package (downloaded by the user or delivered by removable USB media) automatically enrolls the device.

### <a id="1" ></a>Manual enrollment in device management

If you want to use Intune to manage a user device but don't want to join the device to the Azure AD domain, have the user manually enroll the device in device management.

Typically, you would use this method if the user doesn't have an Azure AD license.

After you use this method, the device appears as a Personal device in the **All Devices** list. You can use Intune to apply mobile device management (MDM) policies to the device. However, the device doesn't appear in the **Azure AD Devices** list.

Because the device isn't part of the Azure AD domain, you can't use Azure features such as Conditional Access to manage it. For this reason, we recommend that you don't use this method.

If you do use this method, the user has to follow these steps:

1. On the device, select **Start** > **Settings** > **Accounts** > **Access Work or School**.
2. Select **Enroll only in device management**.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-enroll-device-mgmt-only-2.png" alt-text="Finding the Enroll command in Access work or school settings":::

### <a id="2" ></a>Automatic enrollment by using MDM

This method is one of several that you can use to add a device to an Azure Active Directory domain and then enroll it in Intune.

When you use this method, the user can take either of the following actions:

- Register the device as a Personal device.
- Join the device to the domain as a Corporate device.

After the user makes this choice, the rest of the process is automatic (also known as the "Auto Enroll" process).
After this process is complete, the device appears in both the **All Devices** and **Azure AD Devices** lists as either a Personal or a Corporate device.

Before you try to use this method, make sure of the following conditions:

- The user has an Azure Active Directory Premium license.
- You've configured the MDM scope for your Azure AD environment. For information about how to set up the MDM scope in Azure AD, see [Windows 10, Azure AD and Microsoft Intune: Automatic MDM enrollment powered by the cloud!](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/windows-10-azure-ad-and-microsoft-intune-automatic-mdm/ba-p/244067).

Additionally, for a Corporate device, make sure of the following:

- The device doesn't run Windows 10 Home edition.
- The device isn't joined to an on-premises domain.
- The device isn't already managed by MDM.
- The user who will be signed in to the device has administrator-level permissions on the device.

To be able to use this method, the user has to follow these steps:

1. On the device, select **Start** > **Settings** > **Accounts** > **Access Work or School** > **Connect**.
2. Do one of the following:
   - Register the device as a Personal device. To do this, enter an email address, and then select **Next**.
      :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-auto-enroll-3-personal.png" alt-text="Set up work or school account, enter an email address to register device":::
   - Register the device as a Corporate device. To do this, select **Join this device to Azure Active Directory**.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-auto-enroll-3-corporate.png" alt-text="Set up work or school account, join the device to Azure AD":::

### <a id="3" ></a>Automatic enrollment by using GPO

Both methods that have been discussed to this point require user action to start the enrollment process. Requiring such user activity isn't practical if you have thousands of devices to enroll for your enterprise.

Group Policy provides a means of bulk-enrolling Corporate devices without involving the users (this method doesn't enroll Personal devices). Every device must meet the following conditions:

- The device runs Windows 10, version 1709 or a later version.
- The device is joined to a hybrid Azure AD domain, and has an Azure AD Primary Refresh Token (PRT).

To enroll the devices, configure a Group Policy object (GPO). For more information about how to configure and target the GPO, see [Enroll a Windows 10 device automatically using Group Policy](/windows/client-management/mdm/enroll-a-windows-10-device-automatically-using-group-policy).

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-gpo-for-mem-enrollment-4.png" alt-text="MDM GPO in the Group Policy Management Editor":::

The GPO, in turn, creates scheduled tasks to carry out the enrollment. You can view these tasks in Task Scheduler.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-task-scheduler-5.png" alt-text="Enrollment tasks in Task Scheduler":::

### <a id="4"></a>Windows Autopilot enrollment

The Windows Autopilot enrollment process joins a Corporate device to the Azure AD domain and enrolls it in Intune during the user Out-of-the-box (OOBE) experience. Windows Autopilot is useful if you have to bulk-deploy fresh or refreshed Windows devices.

As part of the Windows Autopilot process, you can customize the greeting screen of the OOBE. You can include company branding and predefined fields and options, as shown in the next figure.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-windows-autopilot-greeting-6.png" alt-text="Customized greeting screen for Windows Autopilot OOBE":::

To configure Windows Autopilot, you have to have the hardware hashes or serial numbers of the device. In Intune, make sure that you configure the MDM scope for Autopilot.
For more information, see [Enroll Windows devices in Intune by using Windows Autopilot](/mem/intune/enrollment/enrollment-autopilot).

> [!NOTE]  
> Windows Autopilot can also join a device to a Hybrid Azure AD domain, and then enroll it in Intune. For more information, see [Deploy hybrid Azure AD-joined devices by using Intune and Windows Autopilot](/mem/intune/enrollment/windows-autopilot-hybrid)

### <a id="5"></a>Intune and Configuration Manager co-management enrollment

You can use this method if you're already using System Center Configuration Manager to manage domain-joined Corporate devices. You can use Configuration Manager to enroll the devices in Intune as Corporate devices. You can then define workloads in Configuration Manager to identify when Configuration Manager policy applies and when Intune policy applies. You don't have to use Group Policy to enroll the devices.

This method requires Configuration Manager version 1710 or a later version.

Every device must meet the following requirements:

- The device runs Windows 10, version 1709 or a later version.
- The device is joined to a hybrid Azure AD domain, and has an Azure AD Primary Refresh Token (PRT).
- The device has a Configuration Manager client installed.

To use this method, you have to add the devices to an appropriate collection in the Configuration Manager console. The following figures show the relevant properties to configure.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-scmm-co-mgmt-props-7a.png" alt-text="Configuration Manager properties for enabling Intune co-management":::

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-scmm-co-mgmt-props-7b.png" alt-text="Configuration Manager properties for co-managing devices in Intune":::

For more information about this enrollment method, see [Support Tip: Understanding auto enrollment in a co-managed environment](https://techcommunity.microsoft.com/t5/intune-customer-success/support-tip-understanding-auto-enrollment-in-a-co-managed/ba-p/834780).

### <a id="6"></a>Deep link enrollment

You can give users a deep link to start the enrollment process for Personal devices.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-deep-link-destination-8a.png" alt-text="Destination of an enrollment deep link":::

This process is the equivalent of the **Enroll only in device management** user interface option.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-deep-link-process-8b.png" alt-text="Deep link enrollment process, step 1":::

The enrollment process starts after the user enters their credentials and selects **Next**.

Users can select a deep link from anywhere in Windows 10. Or you can direct users to an internal web page for enrollment instructions, and include the link on that page.

The link has to have the following URI: **ms-device-enrollment:?mode=mdm**.

The devices have to run Windows 10, version 1607 or a later version.

### <a id="7" ></a>Company portal app or website enrollment

Users don't have to install a company portal app to enroll a Windows 10 device in Intune. However, if you do ask users to install a company portal app (or use a company portal website) you can add device enrollment functions to the portal or website. You can also add the ability to view and install applications that the company has provisioned.

#### Company portal app

If your organization has a company portal app in the Microsoft Store, you can give the following instructions to users:

1. Open Microsoft Store, select **Search** on the menu bar (magnifying glass icon), and then enter the name of the company portal app.
2. In the list of results, select the company portal app, and then select **Get**.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-store-app-9a.png" alt-text="Company portal app in the Microsoft store":::
3. After the app installs, start it, and then sign in when you're prompted. The enrollment process starts automatically.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-store-app-9b.png" alt-text="Signing in to the company portal app":::

#### Company portal website

Provide the following instructions to users so that they can use your company's management portal for enrollment:

1. In a web browser, go to portal.manage.microsoft.com.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-portal-10a.png" alt-text="Example landing page of a company portal website":::
2. Select **Devices**, and then select the gray bar.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-portal-enroll-10b.png" alt-text="Devices page of an example company portal website":::
3. To start the enrollment process, select **Add**.
   :::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-portal-device-10c.png" alt-text="Adding a device by using an example company portal website":::

### <a id="8" ></a>WCD provisioning package enrollment

Windows provisioning functionality provides a method for administrators to seamlessly define configurations for devices, and deploy those configurations to Corporate devices in bulk.

You can use WCD to create a provisioning package (a .ppkg file) that contains configurations settings. In this case, you would create a provisioning package that joins each device to Azure AD and enrolls it in Intune. As shown in the following figure, you can configure packages for different kinds of devices.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-wcd-screen-11.png" alt-text="WCD start page":::

Existing users can access and install the package from their Windows desktop screens. Alternatively, you can load the package on removable USB media and distribute that media to new Windows devices. When a new device starts for the first time, the USB media copies the package to the device. Then the enrollment process is completed during the Windows OOBE experience.

For more information about how to create provisioning packages, see [Bulk enrollment for Windows devices](/mem/intune/enrollment/windows-bulk-enroll).

## Logs

After a Windows device is enrolled in Intune, the following locations on the device contain records of the enrollment process.

- Event Viewer
- MDM diagnostic logs
- Registry

### Event Viewer

The UserDevice Registration and DeviceManagement Enterprise Diagnostics providers generate enrollment-related events. The following table lists some of these events.

|Event Message |Event ID |
| --- | --- |
|Certificate policy request sent successfully |4 |
|Certificate policy response processed successfully |6 |
|Certificate enrollment request sent successfully |8 |
|Certificate enrollment response parsed successfully |10 |
|OMA-DM client configuration succeeds |16 |
|MDM Enroll: OMA-DM client configuration succeeds |16 |
|MDM Enroll: Succeeded |72 |
|Auto MDM Enroll  Succeeded    \|\| (ConfigMgr/GPO) |75 |

The following figures show event information from this table in Event Viewer.
:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-event-viewer-example-12.png" alt-text="Event Viewer, detail view":::

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-event-viewer-example-13.png" alt-text="Event Viewer, overview of enrollment events":::

### MDM diagnostic logs

To see the MDM diagnostic logs on a device, select **Start** > **Settings** > **Accounts** > **Access work or school** > **Info** > **Create report**.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-diagnostics-14a.png" alt-text="Select Create report to see diagnostics":::

To export the diagnostic logs, select **Export**.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-diagnostics-14b.png" alt-text="Select Export to export diagnostics":::

The diagnostics report includes information such as the GUIDs of services that apply policies, and the current values of settings that those policies control.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-diagnostics-report-15.png" alt-text="Diagnostic report information":::

### Registry

To view the enrollment registry entries, open Registry Editor on the device, and locate the following subkey:

> HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Enrollments\C1937429-6DB2-4E10-B99A-A4A1D45C9F3F

The GUID represents the configuration source, as identified in the diagnostics report. The entries and values in this subkey include configuration information and the user principal name (UPN) of the user who is associated with the device.

:::image type="content" source=".\media\enroll-devices-in-intune-tip\intune-registry-16.png" alt-text="Enrollment subkeys and entries in the registry":::

## Comparison

Use the following table to quickly compare the intended devices and user experiences of these eight enrollment methods.

|Enrollment method |Device ownership |User interaction required for enrollment|
| --- | --- | --- |
|Manual (device management only) |Personal |Yes |
|Auto-enroll (MDM) |Personal |Yes |
|Auto-enroll (GPO) |Corporate |No |
|Windows Autopilot |Corporate |Yes |
|Intune/Configuration Manager co-management |Corporate |No |
|Deep link | Personal |Yes |
|Company portal app or website | Personal |Yes |
|WCD provisioning package |Corporate |No |

## Conclusion

Your choice of method to enroll Windows devices in Intune depends on several factors:

- Use case
- Kind of device (Personal or Corporate)
- Available infrastructure (such as System Center Configuration Manager, hardware hash, or an on-premises domain that enforces Group Policy)
- Scale (one personal device, or many enterprise devices)

## References

- [Intune enrollment method capabilities for Windows devices](/mem/intune/enrollment/enrollment-method-capab)
- [MDM enrollment of Windows 10-based devices](/windows/client-management/mdm/mdm-enrollment-of-windows-devices)
