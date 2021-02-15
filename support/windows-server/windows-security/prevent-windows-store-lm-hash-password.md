---
title: Prevent Windows from storing an LM hash of the password in AD and local SAM databases
description: Provides three methods to prevent Windows from storing a LAN manager hash of your password in Active Directory and local SAM databases.
ms.date: 09/08/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Legacy authentication (NTLM)
ms.technology: windows-server-security
---
# How to prevent Windows from storing a LAN manager hash of your password in Active Directory and local SAM databases

This article provides three methods to prevent Windows from storing an LM hash of your password in Active Directory and local SAM databases.

_Original product version:_ &nbsp;Windows 10 - all editions, Windows Server 2012 R2  
_Original KB number:_ &nbsp;299656

## Summary

Instead of storing your user account password in clear-text, Windows generates and stores user account passwords by using two different password representations, generally known as "hashes." When you set or change the password for a user account to a password that contains fewer than 15 characters, Windows generates both a LAN Manager hash (LM hash) and a Windows NT hash (NT hash) of the password. These hashes are stored in the local Security Accounts Manager (SAM) database or in Active Directory.

The LM hash is relatively weak compared to the NT hash, and it's therefore prone to fast brute force attack. Therefore, you may want to prevent Windows from storing an LM hash of your password. This article describes how to do this so that Windows only stores the stronger NT hash of your password.

## More information

Windows 2000-based servers and Windows Server 2003-based servers can authenticate users who connect from computers that are running all earlier versions of Windows. However, versions of Windows earlier than Windows 2000 don't use Kerberos for authentication. For backward compatibility, Windows 2000 and Windows Server 2003 support LAN Manager (LM) authentication, Windows NT (NTLM) authentication, and NTLM version 2 (NTLMv2) authentication. The NTLM, NTLMv2, and Kerberos all use the NT hash, also known as the Unicode hash. The LM authentication protocol uses the LM hash.

It's best to prevent storage of the LM hash if you don't need it for backward compatibility. If your network contains Windows 95, Windows 98, or Macintosh clients, you may experience the following problems if you prevent the storage of LM hashes for your domain:

- Users without an LM hash won't be able to connect to a Windows 95-based computer or a Windows 98-based computer that's acting as a server unless the Directory Services Client for Windows 95 and Windows 98 is installed on the server.
- Users on Windows 95-based computers or Windows 98-based computers won't be able to authenticate to servers by using their domain account unless they have the Directory Services Client installed on their computers.
- Users on Windows 95-based computers or Windows 98-based computers won't be able to authenticate by using a local account on a server if the server has disabled LM hashes unless they have the Directory Services Client installed on their computers.
- Users may not be able to change their domain passwords from a Windows 95-based computer or a Windows 98-based computer, or they may experience account lockout issues when they try to change their passwords from these earlier clients.
- Users of Macintosh Outlook 2001 clients may not be able to access their mailboxes on Microsoft Exchange servers. Users may see the following error in Outlook:
    > The logon credentials supplied were incorrect. Make sure your username and domain are correct, then type your password again.

To prevent Windows from storing an LM hash of your password, use any of the following methods.

### Method 1: Implement the NoLMHash policy by using group policy

To disable the storage of LM hashes of a user's passwords in the local computer's SAM database by using Local Group Policy (Windows XP or Windows Server 2003) or in a Windows Server 2003 Active Directory environment by using Group Policy in Active Directory (Windows Server 2003), follow these steps:

1. In Group Policy, expand **Computer Configuration** > **Windows Settings** > **Security Settings** > **Local Policies**, and then click **Security Options**.
2. In the list of available policies, double-click **Network security: Do not store LAN Manager hash value on next password change**.
3. Click **Enabled** > **OK**.

### Method 2: Implement the NoLMHash policy by editing the registry

In Windows 2000 Service Pack 2 (SP2) and later, use one of the following procedures to prevent Windows from storing an LM hash value on your next password change.

#### Windows 2000 SP2 and Later

> [!IMPORTANT]
> This section, method, or task contains steps that tell you how to modify the registry. However, serious problems might occur if you modify the registry incorrectly. Therefore, make sure that you follow these steps carefully. For added protection, back up the registry before you modify it. Then, you can restore the registry if a problem occurs. For more information about how to back up and restore the registry, click the following article number to view the article in the Microsoft Knowledge Base:
>
> [322756](https://support.microsoft.com/help/322756) How to back up and restore the registry in Windows  
>
> The **NoLMHash** registry key and its functionality were not tested or documented and should be considered unsafe to use in production environments before Windows 2000 SP2.

To add this key by using Registry Editor, follow these steps:

1. Start Registry Editor (Regedt32.exe).
1. Locate and then click the following key:

    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa`
1. On the **Edit** menu, click **Add Key**, type *NoLMHash*, and then press Enter.
1. Quit Registry Editor.
1. Restart the computer, and then change your password to make the setting active.

> [!NOTE]
>
> - This registry key change must be made on all Windows 2000 domain controllers to disable the storage of LM hashes of users' passwords in a Windows 2000 Active Directory environment.
> - This registry key prevents new LM hashes from being created on Windows 2000-based computers, but it doesn't clear the history of previous LM hashes that are stored. Existing LM hashes that are stored will be removed as you change passwords.

#### Windows XP and Windows Server 2003

> [!IMPORTANT]
> This section, method, or task contains steps that tell you how to modify the registry. However, serious problems might occur if you modify the registry incorrectly. Therefore, make sure that you follow these steps carefully. For added protection, back up the registry before you modify it. Then, you can restore the registry if a problem occurs. For more information about how to back up and restore the registry, click the following article number to view the article in the Microsoft Knowledge Base:
>
> [322756](https://support.microsoft.com/help/322756) How to back up and restore the registry in Windows  

To add this DWORD value by using Registry Editor, follow these steps:

1. Click **Start** > **Run**, type *regedit*, and then click **OK**.
2. Locate and then click the following key in the registry:

    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa`

3. On the **Edit** menu, point to **New**, and then click **DWORD Value**.
4. Type NoLMHash, and then press ENTER.
5. On the **Edit** menu, click **Modify**.
6. Type *1*, and then **click OK**.
7. Restart your computer, and then change your password.

> [!NOTE]
>
> - This registry change must be made on all Windows Server 2003 domain controllers to disable the storage of LM hashes of users' passwords in a Windows 2003 Active Directory environment. If you're a domain administrator, you can use Active Directory Users and Computers Microsoft Management Console (MMC) to deploy this policy to all domain controllers or all computers on the domain as described in Method 1 (Implement the NoLMHash Policy by Using Group Policy).
> - This DWORD value prevents new LM hashes from being created on Windows XP-based computers and Windows Server 2003-based computers. The history of all previous LM hashes is cleared when you complete these steps.

> [!IMPORTANT]
> If you're creating a custom policy template that may be used on both Windows 2000 and Windows XP or Windows Server 2003, you can create both the key and the value. The value is in the same place as the key, and a value of 1 disables LM hash creation. The key is upgraded when a Windows 2000 system is upgraded to Windows Server 2003. However, it's okay if both settings are in the registry.

### Method 3: Use a password that's at least 15 characters long

The simplest way to prevent Windows from storing an LM hash of your password is to use a password that is at least 15 characters long. In this case, Windows stores an LM hash value that can't be used to authenticate the user.
