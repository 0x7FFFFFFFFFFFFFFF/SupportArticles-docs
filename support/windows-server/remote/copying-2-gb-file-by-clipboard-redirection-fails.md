---
title: Copying files exceeding 2 GB fails
description: When you try to copy a file larger than 2 GB over a Remote Desktop Services or Terminal Services session through Clipboard Redirection (copy and paste) by using RDP client 6.0 or later, the file fails to copy, and you don't see an error message.
ms.date: 09/08/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Redirection (not printer)
ms.technology: windows-server-rds
---
# Copying files larger than 2 GB over a Remote Desktop Services or Terminal Services session by using Clipboard redirection (copy and paste) fails silently

This article provides help to work around the issue where copying files that's larger than 2 GB over a Remote Desktop Services or Terminal Services session by using Clipboard redirection fails.

_Original product version:_ &nbsp; Windows Server 2012 R2  
_Original KB number:_ &nbsp; 2258090

## Symptoms

When you try to copy a file that is larger than 2 GB over a Remote Desktop Services or a Terminal Services session through Clipboard redirection (copy and paste) by using the Remote Desktop Protocol (RDP) client 6.0 or a later version, the file isn't copied. And, you don't receive an error message.

## Cause

This is a known issue. Copying files larger than 2 GB by using this method isn't supported.

## Resolution

To resolve this issue, use one of the following methods:

- Use Drive Redirection through Remote Desktop Services or a Terminal Services session if you want to transfer files larger than 2 GB.

- Use command-line alternatives such as XCopy to copy files larger than 2 GB over a Remote Desktop Services or Terminal Services session. For example, you can use the following command:

    ```console
    xcopy \\tsclient\c\myfiles\LargeFile d:\temp  
    ```

## More information

- Copy and paste files

    You can copy and paste the files between the remote session and the local computer or between the local computer and the remote session by using the Copy and Paste feature. However, this is limited to files that are smaller than 2 GB.

- Transfer of files by using Drive Redirection

    The drives on the local computer can be redirected in the session so that files can easily be transferred between the local host and the remote computer. The drives that you can use in this manner include the following drivers:

  - local hard disks
  - floppy disk drives
  - mapped network drives

    On the **Local Resources** tab of Remote Desktop Connection, users can specify which kinds of devices and resources that they want to redirect to the remote computer.
