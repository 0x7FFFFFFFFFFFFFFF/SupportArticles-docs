---
title: Devices connected through a Thunderbolt Dock stop working
description: Describes an issue in which devices that are connected through a Thunderbolt Dock stop working after the computer resumes from a power state. Provides a workaround.
ms.date: 11/23/2021
author: Deland-Han
ms.author: delhan
manager: dcscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-client
localization_priority: medium
ms.reviewer: kaushika
ms.custom: sap:power-management, csstroubleshoot
ms.technology: windows-client-deployment
---
# Devices connected through a Thunderbolt Dock stop working after the computer resumes from a power state

This article provides a workaround for an issue where devices that are connected through a Thunderbolt Dock stop working after the computer resumes from a power state.

_Applies to:_ &nbsp; Windows 10 - all editions  
_Original KB number:_ &nbsp; 4532436

## Symptoms

Consider the following scenario:

- On a computer that is running Windows 10 that has a version between Windows 10, version 1709 and Windows 10, version 21H1, you enable Fast Startup.
- On a Thunderbolt Dock, several devices, such as a keyboard, mouse, and USB encryption key, are attached.
- You repeatedly do the following steps:
  - You connect the Thunderbolt Dock to the computer. Devices on the Thunderbolt Dock are enumerated.
  - The system enters or resumes from a Modern Standby, Hibernate (S4) or Soft Off (S5) power state, and you plug or unplug the dock during this period.

In this scenario, the devices stop working and might show yellow exclamation points with the code 24 in the Device Manager.

## Workaround

When the issue occurs, the functionality of the devices can be restored by reattaching the Thunderbolt Dock. If it doesn't work, restart the computer.

## References

For more information about Windows power states, see [System Power States](/windows/win32/power/system-power-states).
