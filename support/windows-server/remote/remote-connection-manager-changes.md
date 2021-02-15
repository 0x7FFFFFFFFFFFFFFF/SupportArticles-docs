---
title: Changes to Remote Connection Manager
description: This article describes the changes that have been made to Remote Connection Manager (RCM) in Windows Server 2016, and explains how to enable RCM.
ms.date: 09/08/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Remote desktop sessions
ms.technology: windows-server-rds
---
# Changes to Remote Connection Manager in Windows Server 2016

This article describes Remote Connection Manager (RCM) and the changes to RCM in Microsoft Windows Server 2016.

_Original product version:_ &nbsp; Windows Server 2016  
_Original KB number:_ &nbsp; 3200967

## Changes to RCM

In Windows Server 2012 R2 and earlier versions, when a user logs on to a terminal server, the RCM contacts the domain controller (DC) to query the configurations that are specific to Remote Desktop on the user object in Active Directory Domain Services (AD DS). This information is displayed in the **Remote Desktop Services Profile** tab of the users object properties in the Active Directory Users and Computers MMC snap-in.

Starting in Windows Server 2016, RCM no longer queries the users object in AD DS. If you require RCM to query AD DS because you are using the Remote Desktop Services attributes, you must manually [enable RCM](#enable-rcm-in-windows-server). For more information about this by-design behavior in Windows Server 2016, see [RCM behavior in Windows Server](#rcm-behavior-in-windows-server).

Additionally, consider the following scenario:

- You install Windows Server 2016 with the Remote Desktop Session Host role.
- You configure a local user account to start an application during logon by using the Local Users and Groups tool in Computer Management.

In this scenario, you expect the user to be presented with the application in the Remote Desktop Session only. However, by default in Remote Desktop Session Host (RDSH) in Windows Server, a full Remote Desktop Session is presented, and the application setup process in the profile doesn't start.

To revert to the earlier (pre-Windows Server 2016) behavior, here's what to do:

If the server has the RD Session Host Role installed, apply the following registry keys to enable the RCM legacy model. This triggers an Active Directory query to check for RDP profile settings:

- Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services` and `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\<Winstation name>\`
- Name: fQueryUserConfigFromDC
- Type: Reg_DWORD
- Value: 1 (Decimal)

Then, restart the Remote Desktop Service. If the server doesn't have the Remote Desktop Service role installed, you must set up an additional registry key:  
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server`

## More information

The attributes that are on the **Remote Desktop Services Profile** tab of the users object properties in AD DS are not applied to the user when the user logs on to a Windows Server Remote Desktop Session Host (RDSH) server. Also, no warning is generated and no event is logged because the user's attributes are not enforced, and because everything is working as designed.

For example, if you are using the RDS attributes to specify a Remote Desktop roaming profile, users will not load that profile but will, instead, use a local profile. When this occurs, there are no error message or logged events. Therefore, the only way that you can know whether the user profile is not the RDS roaming profile is if you notice that the desktop environment is not the expected layout, or if you view the profile types in the system Control Panel applet.

Remote Desktop Services Profile tab in Active Directory Users and Computers:

![Screenshot of the Remote Desktop Services Profile tab](./media/remote-connection-manager-changes/remote-desktop-services-profile-tab.jpg)

User profiles in Control Panel > **System** > **Advanced SystemSettings** > **User Profiles**:

![Screenshot of User profiles](./media/remote-connection-manager-changes/user-profile-in-control-panel.jpg)

The attributes that you can set in the Active Directory Users and Computers MMC snap-in are as follows:

- Profile Path
- Home Folder
- Deny Logon to the RDSH server  

## Enable RCM in Windows Server

> [!IMPORTANT]
> Follow the steps in this section carefully. Serious problems might occur if you modify the registry incorrectly. Before you modify it, [back up the registry for restoration](https://support.microsoft.com/help/322756) in case problems occur.

Use either of the following registry values to enable the behavior of RCM in Windows Servers 2012 R2 and earlier versions:

- Registry key 1:

  - Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services`
  - Name: **fQueryUserConfigFromDC**
  - Type: Reg_DWORD
  - Value: 1 (Decimal)

- Registry key 2:

  - Path: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\<Winstation Name>\`
  - Name: **fQueryUserConfigFromDC**
  - Type: Reg_DWORD
  - Value: 1 (Decimal)

> [!NOTE]
> The default value for \<Winstation Name> is **RDP-tcp**. However, this value can be renamed.

## RCM behavior in Windows Server

At each user logon, RCM does the following things:

- Query the Terminal Services registry key for the **fQueryUserConfigFromDC** value.
- If the value is found and it is set to **1**, contact the DC to get the user configuration information.
- If the value is set to **0** or not present, query the **Winstations** key for the **fQueryUserConfigFromDC** value.
- If the value is found and it is set to **1**, contact the DC to get the user configuration information.
