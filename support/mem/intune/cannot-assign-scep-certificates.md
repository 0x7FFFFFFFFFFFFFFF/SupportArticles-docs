---
title: Can't assign SCEP certificates to devices
description: Fixes an issue in which you can't assign SCEP certificates to devices in Microsoft Intune after you renew an expired certificate.
ms.date: 05/11/2020
ms.prod-support-area-path: Device protection
ms.reviewer: joelste, shhodge
---
# You can't issue SCEP certificates to devices in Intune after a certificate renewal

This article fixes an issue in which you can't assign Simple Certificate Enrollment Protocol (SCEP) certificates to devices in Microsoft Intune after you renew an expired certificate.

_Original product version:_ &nbsp; Microsoft Intune  
_Original KB number:_ &nbsp; 4045957

## Symptoms

You use Microsoft Intune to assign Simple Certificate Enrollment Protocol (SCEP) certificates to devices that you manage. After you renew an expired certificate, new certificates can't be assigned to the devices. When you open the NDESPlugin.log file, the log stops at **Sending request to certificate registration point**.

Additionally, if you [enable CAPI2 logging](/archive/blogs/benjaminperkins/enable-capi2-event-logging-to-troubleshoot-pki-and-ssl-certificate-issues) on the Network Device Enrollment Service (NDES) server, you receive the following error message:

> A required certificate is not within its validity period when verifying against the current system clock or the timestamp in the signed file.

## Cause

This problem occurs because the NDES policy module still uses the thumbprint from an expired client authentication certificate. That certificate was selected when the NDES policy module or Intune Certificate Connector was first installed.

## Resolution

To fix this problem, set the NDES policy module to use the new certificate. To do this, follow these steps on the NDES server:

1. Use `certlm.msc` to open the local computer certificate store, expand **Personal**, and then click **Certificates**.
2. In the list of certificates, find an expired certificate for which the following conditions are true:

   - The value of **Intended Purposes** is **Client Authentication**.
   - The value of **Issued To** or **Common Name** matches the NDES server name.

3. Double-click the certificate to open the **Certificate** dialog box, click the **Details** tab, scroll down to **Thumbprint**, and then verify that the value matches the value of the following registry subkey:

   `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP\Modules\NDESPolicy\NDESCertThumbprint`

4. Click **OK** to close the **Certificate** dialog box, right-click the certificate, and then select **All Tasks** > **Request Certificate with New Key**.
5. In the **Certificate Enrollment** dialog box, click **Next**, and then click **More information is required to enroll for this certificate. Click here to configure settings**.
6. In the **Certificate Properties** dialog box, click the **Subject** tab, and then do the following:

   1. Under **Subject name,** click the **Type** drop-down box and select **Common Name**. In the **Value** box, enter the fully qualified domain name (FQDN) of the NDES server, then click **Add**.
   2. Under **Alternative name**, click the **Type** list, and then select DNS.
   3. In the **Value** box, enter the FQDN of the NDES server, and then click **Add**.

7. Click **OK** to close the **Certificate Properties** dialog box.
8. Click **Enroll**, wait until the enrollment finishes successfully, and then click **Finish**.
9. Double-click the new certificate, and then click the **Details** tab in the **Certificate** dialog box.
10. Scroll down to locate and click **Thumbprint**, and then copy the hexadecimal string from the box.
11. Start Notepad.
12. Paste the hexadecimal string, remove the spaces between the hexadecimal characters, and then save as a text file.

    > [!NOTE]
    > If you receive a warning message about the unicode format, click **OK**.

13. Reopen the text file, copy the thumbprint, and then paste it to the value of the following registry subkey:

    `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\MSCEP\Modules\NDESPolicy\NDESCertThumbprint`

    > [!NOTE]
    > Don't copy any additional characters, such as the question mark at the beginning of the file.

14. At an elevated command prompt, run the following command:

    `iisreset`
