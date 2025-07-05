# SCCM Cloud Management Gateway Failed to get CMG service metadata. Status code is '500' and status description is 'Internal Server Error'

You can find description how I resolved CMG error logged in SMS_CLOUD_PROXCONNECTOR
![01](https://github.com/user-attachments/assets/473ae229-ea24-449d-82ae-d096e4dce6e4)

When I was running _Connection analyzer_<br />
![02](https://github.com/user-attachments/assets/c5b9900f-a153-4941-9e9e-41fdc6f7ddda)

Configuration version of the CMG service should be 3. Failed to get CMG service metadata. Status code is '500' and status description is 'Internal Server Error'...<br />
![03](https://github.com/user-attachments/assets/6b9edce4-00cf-4250-8e95-5696b9b9261d)

_Connection status_ of _Connection Point Servers_ was in 'Disconnected' state
![04](https://github.com/user-attachments/assets/ad533769-eccf-4110-be91-1e0d446c52e0)

After quite some resarch most forums and documentation pointed to the possible issues with Certificates and/or DNS (CNAME records) and/or Networking.
Nothing was making sense in the logs, so I decided to check Azure side of the Cloud Management Gateway.

There I found that VMSS instance was in **Unhealthy** state 
![05](https://github.com/user-attachments/assets/d1ec4b02-78b2-44e5-abfc-fbea840b629a)

Tried to reboot Virtual machine, it failed <br />
![06](https://github.com/user-attachments/assets/3dcbd76f-a5be-42c5-bceb-be86fa24e3a9)

Decided to reinstall _Microsoft Powershell Desired State Configuration_ extension as it was preventing Virtual machines from booting

---
> [!CAUTION]
> **Make sure to save configuration settings before uninstalling extension**<br />
> VMSS - Extensions - Microsoft.Powershell.DSC - Settings
---

While extension was uninstalled, logically we can see that _Connection analyzer_ is unable to connect to the CMG service<br />
![07](https://github.com/user-attachments/assets/6721a9b2-b0fa-4b75-8a7e-131df79407ea)

> [!TIP]
> You can deploy the extension through the portal by adjusting the syntax. However, I opted for the .json format since the settings I copied were already in that format.<br />
> ARM deployment template https://github.com/jazeps-vy/SCCM-CMG-Status-Code-500/blob/main/vmss-ps-dsc-extension.json

Replace 'configuration' and 'configurationArguments' before saving ARM template<br />
Generate temprorary SAS token for resource deployment (Blob containers - path-from-configuration - Settings - Shared access tokens)<br />
Enter parameters VMSS name, ArchiveFileName (script name from configuration "url"), SAS token and ExtensionTagVersion (I used 2.77)<br />
![08](https://github.com/user-attachments/assets/d94fe9d5-4814-4102-aaa4-d963ab193b5d)

SCCM - Restarted CMG service, _Connection status_ of _Connection Point Servers_ was 'Connected'<br />
![09](https://github.com/user-attachments/assets/aa186dcf-5fdc-4251-b4ef-35837ea3bb13)

Triggered _Synchronize configuration_, VMSS instances went in to the 'Updating' state
![10](https://github.com/user-attachments/assets/b294b471-254f-4ccf-9685-3a971d524a17)

Problem solved! PROXYCONNECTOR started to log successful client connections
![11](https://github.com/user-attachments/assets/31ff9b44-5814-41cd-a2d0-bcd645540b48)
