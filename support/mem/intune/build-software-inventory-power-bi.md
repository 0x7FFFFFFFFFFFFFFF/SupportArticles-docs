---
title: Build software inventory using Power BI
description: Describes how to build software inventory by using Power BI to connect to the Intune Data Warehouse.
author: helenclu
ms.author: luche
ms.reviewer: samustaf
ms.date: 10/10/2020
ms.custom: sap:app management
---
# Support Tip: Build software inventory using Power BI

[Intune discovered apps](/mem/intune/apps/app-discovered-apps) is a list of apps that are detected on the Intune-enrolled devices in your tenant. It acts as a software inventory for your tenant. In general, the report refreshes every seven days from the time of enrollment (not a weekly refresh for the entire tenant). 

By using the [Intune Data Warehouse](/mem/intune/developer/reports-nav-create-intune-reports), you can refresh the data on a daily 
cadence. This article discusses how to build the software inventory by using Power BI.

## Install Power BI Desktop

1. On your Windows 10 desktop, start the **Microsoft Store** app, and then search for **Power BI Desktop**.
2. Select **Get**.

   :::image type="content" source="media/build-software-inventory-power-bi/download-power-bi-desktop.png" alt-text="Download Power BI Desktop from Microsoft Store.":::

3. After the installation finishes, select **Launch**.

## Connect Power BI Desktop to the Intune Data Warehouse

Follow step 1 to 10 in [Connect to the OData feed for the Intune Data Warehouse for your tenant](/mem/intune/developer/reports-proc-create-with-odata#connect-to-the-odata-feed-for-the-intune-data-warehouse-for-your-tenant). Select the required tables to build your report, and then select **Load**.

> [!NOTE]
> There are two versions of the Intune Data Warehouse: 
>  
> - To use version 1.0, set the query parameter `api-version=v1.0` in the custom feed URL.
> - To use the beta version, set the query parameter `api-version=beta` in the custom feed URL.

## Build the report

1. In the **Visualizations** pane, select the ellipsis button (**...**), and then select **Import from marketplace**.

   :::image type="content" source="media/build-software-inventory-power-bi/import-from-marketplace.png" alt-text="Import from marketplace in the Visualizations pane.":::
2. In the Power BI Visuals window, select **Filters**, locate **HierarchySlicer**, and then select **Add**.

   :::image type="content" source="media/build-software-inventory-power-bi/hierarchy-slicer.png" alt-text="Add HierarchySlicer.":::

3. In the **Visualizations** pane, select **HierarchySlicer** to add it to the report canvas.
4. In the **Fields** pane, locate and expand the `applicationInventories` table. Select the `applicationName` and `applicationVersion` data fields, drag the data fields to the **Visualizations** pane, and then drop them into the **Values** section in the box that's labeled **Add data fields here**.

   :::image type="content" source="media/build-software-inventory-power-bi/add-application-fields.png" alt-text="Add applicationName and applicationVersion." lightbox="media/build-software-inventory-power-bi/add-application-fields.png":::

   By adding multiple data fields, you can drill down within these values. For example, by adding the application name and application version, you can search the required application based on versions.

5. To find a specific application, select the ellipsis button (**...**), and then select **Search**.

   :::image type="content" source="media/build-software-inventory-power-bi/enable-search.png" alt-text="Search for an application.":::

6. To create a list of devices, select **Table** in the **Visualizations** pane.
7. In the **Fields** pane, locate the `devices` table, select `deviceName`, `operatingSystem`, `osVersion`, and `isDeleted`. From the `user` table, select `userPrincipalName`. From the `applicationInventories` table, select `applicationName`.

   :::image type="content" source="media/build-software-inventory-power-bi/devices.png" alt-text="Device table in the Fields pane.":::
 
8. To filter the devices based on specific installed apps, search for the app in the hierarchy slicer. For example, the following screenshot shows the devices that have Outlook installed.

   :::image type="content" source="media/build-software-inventory-power-bi/filter-devices.png" alt-text="Filter devices based on apps.":::

9. You can also use [the Q&A visual](/power-bi/visuals/power-bi-visualization-q-and-a) to ask natural language questions and get answers in the form of a visualization.







 
