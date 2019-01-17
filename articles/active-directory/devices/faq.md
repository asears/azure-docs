---
title: Azure Active Directory device management FAQ | Microsoft Docs
description: Azure Active Directory device management FAQ.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman

ms.assetid: cdc25576-37f2-4afb-a786-f59ba4c284c2
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.author: markvi
ms.reviewer: jairoc

---

# Azure Active Directory device management FAQ

**Q: I registered the device recently. Why can’t I see the device under my user info in the Azure portal? Or Why is device owner marked as N/A for hybrid Azure AD joined devices?**
**A:** Windows 10 devices that are hybrid Azure AD joined do not show up under the USER devices.
You need to use All devices view in Azure portal. You can also use PowerShell [Get-MsolDevice](/powershell/module/msonline/get-msoldevice?view=azureadps-1.0) cmdlet.

Only the following devices are listed under the USER devices:

- All personal devices that are not hybrid Azure AD joined. 
- All non-Windows 10 / Windows Server 2016 devices.
- All non-Windows devices 

--- 

**Q: How do I know what the device registration state of the client is?**

**A:** You can use the Azure portal, go to All devices and search for the device using device ID. Check the value under the join type column. Sometimes, the device could have been reset or reimaged. So, it is essential to also check device registration state on the device too:

- For Windows 10 and Windows Server 2016 or later devices, run dsregcmd.exe /status.
- For down-level OS versions, run "%programFiles%\Microsoft Workplace Join\autoworkplace.exe"

---

**Q: I see the device record under the USER info in the Azure portal and can see the state as registered on the device. Am I set up correctly for using conditional access?**

**A:** The device join state, reflected by deviceID, must match with that on Azure AD and meet any evaluation criteria for conditional access. 
For more information, see [Require managed devices for cloud app access with conditional access](../conditional-access/require-managed-devices.md).

---

**Q: I have deleted in the Azure portal or using Windows PowerShell, but the local state on the device says that it is still registered?**

**A:** This is by design. The device will not have access to resources in the cloud. 

If you want to re-register again, a manual action must be to be taken on the device. 

To clear the join state from Windows 10 and Windows Server 2016 that are on-premises AD domain-joined:

1.	Open the command prompt as an administrator.

2.	Type `dsregcmd.exe /debug /leave`

3.	Sign out and sign in to trigger the scheduled task that registers the device with Azure AD again. 

For down-level Windows OS versions that are on-premises AD domain-joined:

1.	Open the command prompt as an administrator.
2.	Type `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe /l"`.
3.	Type `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe /j"`.

---

**Q: Why do I see duplicate device entries in Azure portal?**

**A:**

-	For Windows 10 and Windows Server 2016, if there are repeated attempts to unjoin and rejoin the same device, there might be duplicate entries. 

-	If you have used Add Work or School Account, each windows user who uses Add Work or School Account will create a new device record with the same device name.

-	For down-level Windows OS versions that are on-premises AD domain-joined using automatic registration will create a new device record with the same device name for each domain user who logs into the device. 

-	An Azure AD joined machine that has been wiped, reinstalled, and rejoined with the same name, will show up as another record with the same device name.

---

**Q: Why can a user still access resources from a device I have disabled in the Azure portal?**

**A:** It can take up to an hour for a revoke to be applied.

>[!Note] 
>For enrolled devices, we recommend wiping the device to ensure that users cannot access the resources. For more information, see [Enroll devices for management in Intune](https://docs.microsoft.com/intune/deploy-use/enroll-devices-in-microsoft-intune). 

---

## Azure AD Join FAQ

**Q: How do I unjoin an Azure AD Joined device locally on the device?**

**A:** 
- For hybrid Azure AD Joined devices, make sure to turn off auto registration so that the scheduled task does not register the device again. Next, open command prompt as an administrator and type `dsregcmd.exe /debug /leave`. Alternatively, this command can be run as a script across multiple devices to unjoin in bulk.

- For pure Azure AD Joined devices, make sure you have an offline local administrator account or create one, as you won't be able to sign in with any Azure AD user credentials. Next, go to **Settings** > **Accounts** > **Access Work or School**. Select your account and click on **Disconnect**. Follow the prompts and provide the local administrator credentials when prompted. Reboot the device to complete the unjoin process.

---

**Q: Can my users sign in to Azure AD joined devices that have been deleted or disabled in Azure AD?**
**A:** Yes. Windows has cached logon capability to allow previously logged in users to access desktop quickly even without network connectivity. When a device is deleted or disabled in Azure AD, it is not known to the Windows device. So, previously logged in users can continue to access the desktop with cached logon. However, as the device is deleted or disabled, users cannot access any resources protected by device-based Conditional Access. 

Users who haven't already logged in cannot access the device as there is no cached logon enabled for them. 

---

**Q: Can disabled or deleted users sign in to Azure AD joined devices?**
**A:** Yes, but only for a limited time. When a user is deleted or disabled in Azure AD, it is not immediately known to the Windows device. So, previously logged in users can access the desktop with cached logon. Once the device is aware of the user state (typically in less than 4 hours), Windows blocks those users from accessing the desktop. As the user is deleted or disabled in Azure AD, all their tokens will be revoked, so they cannot access any resources. 

Deleted or disabled users who haven't logged in previously cannot access a device as there is no cached logon enabled for them. 

---

**Q: My users cannot search printers from Azure AD Joined devices. How can I enable printing from Azure AD Joined devices?**

**A:** For deploying printers for Azure AD Joined devices, see [Hybrid cloud print](https://docs.microsoft.com/windows-server/administration/hybrid-cloud-print/hybrid-cloud-print-deploy). You will need an on-premises Windows Server to deploy hybrid cloud print. Currently, cloud-based print service is not available. 

---

**Q: How do I connect to a remote Azure AD joined device?**
**A:** Refer to the article https://docs.microsoft.com/windows/client-management/connect-to-remote-aadj-pc for details.

---

**Q: Why do my users see “You can’t get there from here”?**

**A:** If you have configured certain conditional access rules to require a specific device state and the device does not meet the criteria, users are blocked and see this message. 
Evaluate the conditional access policy rules and ensure that the device is able to meet the criteria to avoid this message.

---

**Q: Why do some of my users do not get MFA prompts on Azure AD joined devices?**

**A:** If user joins or registers a device with Azure AD using multi-factor auth, the device itself will become a trusted second factor for that particular user. Subsequently, whenever the same user signs in to the device and accesses an application, Azure AD considers the device as a second factor and enables that user to seamlessly access their applications without additional MFA prompts. This behavior is not applicable to any other user signing into that device, so all other users accessing that device would still be prompted with an MFA challenge before accessing applications that require MFA.

---

**Q: Why do I get a "username or password is incorrect" message for a device I have just joined to Azure AD?**

**A:** Common reasons for this scenario are:

- Your user credentials are no longer valid.

- Your computer is unable to communicate with Azure Active Directory. Check for any network connectivity issues.

- Federated logins require your federation server to support WS-Trust endpoints enabled and accessible. 

- You have enabled Pass through Authentication and the user has a temporary password that needs to be changed on logon.

---

**Q: Why do I see the “Oops… an error occurred!" dialog when I try do Azure AD join my PC?**

**A:** This is a result of setting up Azure Active Directory enrollment with Intune. Make sure that the user attempting to do Azure AD join has correct Intune license assigned. For more information, see [Set up Windows device management](https://docs.microsoft.com/intune/deploy-use/set-up-windows-device-management-with-microsoft-intune#azure-active-directory-enrollment).  

---

**Q: Why did my attempt to Azure AD join a PC fail although I didn't get any error information?**

**A:** A likely cause is that the user is logged in to the device using the local built-in administrator account. 
Create a different local account before using Azure Active Directory Join to complete the setup. 

---

## Hybrid Azure AD Join FAQ

**Q: Where can I find troubleshooting information for diagnosing hybrid Azure AD join failures?**

**A:** For troubleshooting information, see:

- [Troubleshooting autoregistration of domain joined computers to Azure AD – Windows 10 and Windows Server 2016](troubleshoot-hybrid-join-windows-current.md)

- [Troubleshooting autoregistration of domain joined computers to Azure AD for Windows down-level clients](troubleshoot-hybrid-join-windows-legacy.md)
 
**Q: Why do I see a duplicate Azure AD registered record for my Windows 10 hybrid Azure AD joined device in the Azure AD devices list?**

**A:** When your users add their account to apps on a domain joined device, they could be prompted to "Add account to Windows?". Clicking "Yes" on the prompt would lead to the device to be registered with Azure AD and the trust type marked as Azure AD registered. Once you enable Hybrid Azure AD join in your organization, the device will also get hybrid Azure AD joined. As a result, there will be two device states showing up for the same device. However, hybrid Azure AD join takes precedence over Azure AD registered state. So your device will be considered hybrid Azure AD join for any authentication and Conditional Access evaluation. So, you can safely delete the Azure AD registered device record from the Azure AD portal. Please review [this section](https://docs.microsoft.com/azure/active-directory/devices/hybrid-azuread-join-plan#review-things-you-should-know) in the Hybrid Azure AD join article to understand how to avoid or cleanup this dual state on the Windows 10 machine. 

---

## Azure AD Register FAQ

**Q: Can I register Android or iOS BYOD devices?**

**A:** Yes, but only with Azure device registration service and for hybrid customers. It is not supported with on-premises device registration service in AD FS.

**Q: How can I register a macOS device?**

**A:** To register macOS device:

1.	[Create a compliance policy](https://docs.microsoft.com/intune/compliance-policy-create-mac-os)
2.	[Define a conditional access policy for macOS devices](../active-directory-conditional-access-azure-portal.md) 

**Remarks:**

- The users that are included in your conditional access policy need a [supported version of Office for macOS](../conditional-access/technical-reference.md#client-apps-condition) to access resources. 

- During the first access attempt, your users are prompted to enroll the device using the company portal.

---
