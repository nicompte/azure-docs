---
title: Support matrix for Arc-enabled VMware vSphere (preview)
description: In this article, you'll learn about the support matrix for Arc-enabled VMware vSphere including vCenter Server versions supported, network requirements etc.
ms.topic: how-to 
ms.date: 09/30/2022

# Customer intent: As a VI admin, I want to understand the support matrix for Arc-enabled VMware vSphere.
---

# Support matrix for Arc-enabled VMware vSphere (preview)

This article documents the prerequisites and support requirements for using the [Arc-enabled VMware vSphere (preview)](overview.md) to manage your VMware vSphere VMs through Azure Arc.

To use Arc-enabled VMware vSphere, you must deploy an Azure Arc resource bridge in your VMware vSphere environment. The resource bridge provides an ongoing connection between your VMware vCenter Server and Azure. Once you've connected your VMware vCenter Server to Azure, components on the resource bridge discover your vCenter inventory. You can enable them in Azure and start performing virtual hardware and guest OS operations on them using Azure Arc.


## VMware vSphere Requirements

### Supported vCenter Server versions

- vCenter Server version 6.7 or 7.

### Required vSphere account privileges

You need a vSphere account that can:
- Read all inventory. 
- Deploy and update VMs to all the resource pools (or clusters), networks, and VM templates that you want to use with Azure Arc.

This account is used for the ongoing operation of Azure Arc-enabled VMware vSphere (preview) and the deployment of the Azure Arc resource bridge (preview) VM.

### Resource bridge resource requirements 

For Arc-enabled VMware vSphere, resource bridge has the following minimum virtual hardware requirements

- 16 GB of memory
- 4 vCPUs
- An external virtual switch that can provide access to the internet directly or through a proxy. If internet access is through a proxy or firewall, ensure [these URLs](#resource-bridge-networking-requirements) are allow-listed.

### Resource bridge networking requirements

The following firewall URL exceptions are needed for the Azure Arc resource bridge VM:

| **Service** | **Port** | **URL** | **Direction** | **Notes**|
| --- | --- | --- | --- | --- |
| Microsoft container registry | 443 | https://mcr.microsoft.com | Appliance VM IP and control plane endpoint need outbound connection. | Required to pull container images for installation. |
| Azure Arc Identity service | 443 | https://*.his.arc.azure.com | Appliance VM IP and control plane endpoint need outbound connection. | Manages identity and access control for Azure resources |
| Azure Arc configuration service | 443	| https://*.dp.kubernetesconfiguration.azure.com | Appliance VM IP and control plane endpoint need outbound connection. | Used for Kubernetes cluster configuration. |
| Cluster connect service | 443	| https://*.servicebus.windows.net | Appliance VM IP and control plane endpoint need outbound connection. | Provides cloud-enabled communication to connect on-premises resources with the cloud. |
| Guest Notification service | 443 | https://guestnotificationservice.azure.com	| Appliance VM IP and control plane endpoint need outbound connection. | Used to connect on-premises resources to Azure. |
| SFS API endpoint | 443 | msk8s.api.cdp.microsoft.com | Host machine, Appliance VM IP and control plane endpoint need outbound connection. | Used when downloading product catalog, product bits, and OS images from SFS. |
| Resource bridge (appliance) Dataplane service | 443 | https://*.dp.prod.appliances.azure.com | Appliance VM IP and control plane endpoint need outbound connection. | Communicate with resource provider in Azure. |
| Resource bridge (appliance) container image download | 443 | *.blob.core.windows.net, https://ecpacr.azurecr.io | Appliance VM IP and control plane endpoint need outbound connection. | Required to pull container images. |
| Resource bridge (appliance) image download | 80 | *.dl.delivery.mp.microsoft.com | Host machine, Appliance VM IP and control plane endpoint need outbound connection. | Download the Arc resource bridge OS images. |
| Azure Arc for K8s container image download | 443 | https://azurearcfork8sdev.azurecr.io | Appliance VM IP and control plane endpoint need outbound connection. | Required to pull container images. |
| ADHS telemetry service | 443 | adhs.events.data.microsoft.com  | Appliance VM IP and control plane endpoint need outbound connection.	Runs inside the appliance/mariner OS. | Used periodically to send Microsoft required diagnostic data from control plane nodes. Used when telemetry is coming off Mariner, which would mean any K8s control plane. |
| Microsoft events data service | 443 | v20.events.data.microsoft.com  | Appliance VM IP and control plane endpoint need outbound connection. | Used periodically to send Microsoft required diagnostic data from the Azure Stack HCI or Windows Server host. Used when telemetry is coming off Windows like Windows Server or HCI. |
| vCenter Server | 443 | URL of the vCenter server  | Appliance VM IP and control plane endpoint need outbound connection. | Used to by the vCenter server to communicate with the Appliance VM and the control plane.|

## Azure permissions required

Following are the minimum Azure roles required for various operations:

| **Operation** | **Minimum role required** | **Scope** |
| --- | --- | --- |
| Onboarding your vCenter Server to Arc | Azure Arc VMware Private Clouds Onboarding | On the subscription or resource group into which you want to onboard |
| Administering Arc-enabled VMware vSphere | Azure Arc VMware Administrator | On the subscription or resource group where vCenter server resource is created |
| VM Provisioning | Azure Arc VMware Private Cloud User | On the subscription or resource group that contains the resource pool/cluster/host, datastore and virtual network resources, or on the resources themselves |
| VM Provisioning | Azure Arc VMware VM Contributor | On the subscription or resource group where you want to provision VMs |
| VM Operations | Azure Arc VMware VM Contributor | On the subscription or resource group that contains the VM, or on the VM itself |

Any roles with higher permissions such as *Owner/Contributor* role on the same scope, will also allow you to perform all the operations listed above.

## Guest management (Arc agent) requirements

With Arc-enabled VMware vSphere, you can install the Arc connected machine agent on your VMs at scale and use Azure management services on the VMs. There are additional requirements for this capability:

To enable guest management (install the Arc connected machine agent), ensure

- VM is powered on
- VM has VMware tools installed and running
- Resource bridge has access to the host on which the VM is running
- VM is running a [supported operating system](#supported-operating-systems)
- VM has internet connectivity directly or through proxy. If the connection is through a proxy, ensure [these URLs](#networking-requirements) are allow-listed.

### Supported operating systems

The officially supported versions of the Windows and Linux operating system for the Azure Connected Machine agent are listed [here](../servers/prerequisites.md#supported-operating-systems). Only x86-64 (64-bit) architectures are supported. x86 (32-bit) and ARM-based architectures, including x86-64 emulation on arm64, aren't supported operating environments.

### Software requirements

Windows operating systems:

* NET Framework 4.6 or later is required. [Download the .NET Framework](/dotnet/framework/install/guide-for-developers).
* Windows PowerShell 5.1 is required. [Download Windows Management Framework 5.1.](https://www.microsoft.com/download/details.aspx?id=54616).

Linux operating systems:

* systemd
* wget (to download the installation script)

### Networking requirements

The following firewall URL exceptions are needed for the Azure Arc agents:

| **URL** | **Description** |
| --- | --- |
| aka.ms | Used to resolve the download script during installation |
| download.microsoft.com | Used to download the Windows installation package |
| packages.microsoft.com | Used to download the Linux installation package |
| login.windows.net | Azure Active Directory |
| login.microsoftonline.com | Azure Active Directory |
| pas.windows.net | Azure Active Directory |
| management.azure.com | Azure Resource Manager - to create or delete the Arc server resource |
| *.his.arc.azure.com | Metadata and hybrid identity services |
| *.guestconfiguration.azure.com | Extension management and guest configuration services |
| guestnotificationservice.azure.com, *.guestnotificationservice.azure.com | Notification service for extension and connectivity scenarios |
| azgn*.servicebus.windows.net | Notification service for extension and connectivity scenarios |
| *.servicebus.windows.net | For Windows Admin Center and SSH scenarios |
| *.blob.core.windows.net | Download source for Azure Arc-enabled servers extensions |
| dc.services.visualstudio.com | Agent telemetry |


## Next steps

- [Connect VMware vCenter to Azure Arc using the helper script](quick-start-connect-vcenter-to-arc-using-script.md)
