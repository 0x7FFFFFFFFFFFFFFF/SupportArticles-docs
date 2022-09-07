---
<<<<<<< HEAD
title: The certificate chain was issued by an authority that isn’t trusted, error after upgrading SNAC applications.
=======
title: The certificate chain was issued by an authority that isn’t trusted
>>>>>>> 7f95a4c07c1f2f9f380209a95a78582dd694295e
description: This article provides resolutions for the error that occurs when you upgrade SNAC applications.
ms.date: 07/09/2022
author: ramakoni
ms.author: v-jayaraman-p
ms.custom: sap:Connection issues
ms.prod: sql
---

# "The certificate chain was issued by an authority that isn’t trusted" error after upgrading SNAC applications

Support for the SQL Server Native Client 11.0 (SNAC) as a driver for database applications ended on July 12, 2022. Any applications that use the SNAC 11.0 must be updated to use newer versions of the drivers (see [Download ODBC Driver for SQL Server](/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16&preserve-view=true) - ODBC Driver for SQL Server and [Download Microsoft OLE DB Driver for SQL Server](/sql/connect/oledb/download-oledb-driver-for-sql-server?view=sql-server-ver16&preserve-view=true) - OLE DB Driver for SQL Server). This article describes an issue that occurs when you upgrade your SNAC 11.0 application to use these new drivers.

## Error messages

**Scenario 1: Applications upgraded to Microsoft OLE DB Driver 19 for SQL Server**

If you recently upgraded your SQL Server Native Client 11.0 (Provider=SQLNCLI11) application to use Microsoft OLE DB Driver 19 for SQL Server (Provider=MSOLEDBSQL19), you might receive error messages that resemble the following messages:

```sql

[Microsoft OLE DB Driver 19 for SQL Server]: Client unable to establish connection 

[Microsoft OLE DB Driver 19 for SQL Server]: SSL Provider: The certificate chain was issued by an authority that is not trusted.

```

**Scenario 2: Applications upgraded to Microsoft ODBC Driver 18.*x* for SQL Server**

If you recently upgraded your SQL Server Native Client 11.0 (Driver={SQL Server Native Client 11.0}) application to Microsoft ODBC Driver 18 for SQL Server (Driver={ODBC Driver 18 for SQL Server}), you might receive error messages that resemble the following messages:

```sql

[Microsoft][ODBC Driver 18 for SQL Server]SSL Provider: The certificate chain was issued by an authority that is not trusted.

[Microsoft][ODBC Driver 18 for SQL Server]Client unable to establish connection

```

## Cause

These errors occur if both the following conditions are true:

- The **Force encryption** setting for the SQL Server instance is set to **No**.

- The client connection string doesn’t explicitly specify a value for encryption property, or the **Encryption** option wasn't explicitly set or updated in the DSN.

The error occurs because of a change in the default behavior of the client drivers. Older versions of client drivers are designed to assume that data encryption is **OFF** by default. The new drivers assume this setting to be **ON** by default. Because data encryption is set to **ON**, the driver tries to validate the server’s certificate and fails. When it fails, the driver displays the following error message in the [Error messages](#error-messages) section:

`The certificate chain was issued by an authority that is not trusted`

## Resolutions

For scenario 1, use one of the following solutions for short-term mitigation:

- **Solution 1:** Use Microsoft OLE DB Driver for SQL Server 18.x. You can download the driver from [Release notes for OLE DB Driver - OLE DB Driver for SQL Server](/sql/connect/oledb/release-notes-for-oledb-driver-for-sql-server?view=sql-server-ver16&preserve-view=true).

- **Solution 2:** If the application connection string property already specifies a value of **Yes** or **Mandatory** for the **Encrypt/Use Encryption for Data setting**, change the value to **No** or **Optional**. For example, **Use Encryption for Data=Optional**. If the connection string doesn’t specify any value for **Encrypt/Use Encryption for Data**, add **Use Encryption for Data=Optional** to the connection string. For more information, see [Encryption and certificate validation - OLE DB Driver for SQL Server](/sql/connect/oledb/features/encryption-and-certificate-validation?view=sql-server-ver16&preserve-view=true).

- **Solution 3:** Add `;Trust Server Certificate=true` to the connection string. This will force the client to trust the certificate without validation.

For scenario 2, use one of the following solutions for short-term mitigation:

- **Solution 1:** Use the Microsoft ODBC Driver 17 for SQL Server. You can download the driver from [Download ODBC Driver for SQL Server](/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16&preserve-view=true).

- **Solution 2:** If the application connection string property already specifies a value of **Yes** or **Mandatory for Encrypt** setting, change the value to **No** or **Optional**. If the value isn’t already specified, add `Encrypt = Optional;`. If you’re using a DSN, change the encryption setting from **Mandatory** to **Optional**. For more information, see [ODBC DSN and connection string keywords - ODBC Driver for SQL Server](/sql/connect/odbc/dsn-connection-string-attribute?view=sql-server-ver16&preserve-view=true).

## See also

- [Enable encrypted connections - SQL Server](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine?view=sql-server-ver16&preserve-view=true)

- [The certificate received from the remote server was issued by an untrusted certificate authority error when you connect to SQL Server](error-message-when-you-connect.md)

- [Support Policies - SQL Server Native Client](/sql/relational-databases/native-client/applications/support-policies-for-sql-server-native-client?view=sql-server-ver16&preserve-view=true)

- [SNAC lifecycle explained - Microsoft Tech Community](https://techcommunity.microsoft.com/t5/sql-server-blog/snac-lifecycle-explained/ba-p/385381)
