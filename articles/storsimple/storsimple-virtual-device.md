<properties
   pageTitle="StorSimple virtual device in Azure | Microsoft Azure"
   description="Learn how to create, deploy, and manage a StorSimple virtual device in a Microsoft Azure virtual network. (Applies to StorSimple version .3 and earlier.)"
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carolz"
   editor="" />
<tags
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="12/01/2015"
   ms.author="alkohli" />

# Deploy and manage a StorSimple virtual device in Azure

> [AZURE.SELECTOR]
- [Update 2](../articles/storsimple/storsimple-virtual-device-u2.md)
- [Update 1](../articles/storsimple/storsimple-virtual-device-u1.md)
- [GA Release](../articles/storsimple/storsimple-virtual-device.md)

## Overview

The StorSimple virtual device is an additional capability that comes with your Microsoft Azure StorSimple solution. The StorSimple virtual device runs on a virtual machine in a Microsoft Azure virtual network, and you can use it to back up and clone data from your hosts. The following topics in this article will help you learn about, configure, and use the StorSimple virtual device.

- How the virtual device differs from the physical device.

- Security considerations for using a virtual device.

- Prerequisites for the virtual device.

- Create and configure the virtual device.

- Work with the virtual device.

- Failover to the virtual device.

- Shut down or delete the virtual device.


## How the virtual device differs from the physical device

The StorSimple virtual device is a software-only version of StorSimple that runs on a single node in a Microsoft Azure virtual machine. The virtual device supports disaster recovery scenarios in which your physical device is not available, and is appropriate for use in cloud dev and test scenarios.

The following are some key differences between the StorSimple virtual device and the physical StorSimple device:

- The virtual device has only one network interface: DATA 0. The physical device has six network interfaces: DATA 0 through DATA 5.
- The virtual device is registered during the configuration step instead of as a separate task.
- The service data encryption key cannot be regenerated from the virtual device. During key rollover, you will regenerate the key on the physical device and then update the virtual device with the new key.
- If updates need to be applied to the virtual device, it will experience some down time, whereas the physical device will not.

## Security considerations for using a virtual device

Keep the following security considerations in mind when you use the StorSimple virtual device:

- The virtual device is secured through your Microsoft Azure subscription. This means that if you are using the virtual device and your Azure subscription is compromised, the data stored on your virtual device is also susceptible.

- The public key of the certificate used to encrypt data stored in Azure StorSimple is securely made available to the Azure classic portal, and the private key is retained with the StorSimple device. On the StorSimple virtual device, both the public and private keys are stored in Azure.

- The virtual device is hosted in the Microsoft Azure datacenter.


## Prerequisites for the virtual device

The following sections will help you prepare to use the StorSimple virtual device.

### Azure requirements

Before you provision the virtual device, you need to make the following preparations in your Azure environment:

- For the virtual device, [configure a virtual network on Azure](../virtual-network/virtual-networks-create-vnet-classic-portal.md).
- It is advisable to use the default DNS server provided by Azure instead of specifying your own DNS server name. If your DNS server name is not valid, the creation of the virtual device will fail.
- Point-to-site and site-to-site are optional, but not required. If you wish, you can configure these options for more advanced scenarios.

>[AZURE.IMPORTANT] **Make sure that the virtual network is in the same region as the cloud storage accounts that you are going to be using with the virtual device.**

- You can create [Azure Virtual Machines](../virtual-machines/virtual-machines-linux-about.md) (host servers) in the virtual network that can use the volumes exposed by the virtual device. These servers must meet the following requirements:
	- Be Windows or Linux VMs with iSCSI Initiator software installed.
	- Be running in the same virtual network as the virtual device.
	- Be able to connect to the iSCSI target of the virtual device through the internal IP address of the virtual device.

- Make sure you have configured support for iSCSI and cloud traffic on the same virtual network.

### StorSimple requirements

Make the following updates to your Azure StorSimple service before you create a virtual device:


- Add [access control records](storsimple-manage-acrs.md) for the VMs that are going to be host servers for your virtual device.

- Make sure that you have a [storage account](storsimple-manage-storage-accounts.md#add-a-storage-account) in the same region as the virtual device. Storage accounts in different regions may result in poor performance.

- Make sure that you use a different storage account for virtual device creation from the one used for your data. Using the same storage account may result in poor performance.

Make sure that you have the following information before you begin:

- You have your Azure classic portal account with access credentials.

- You have your Azure storage account access credentials.

- You have a copy of the service data encryption key from your physical device.

- You have a copy of the cloud service encryption key for each volume container.


## Create and configure the virtual device

Before performing these procedures, make sure that you have met the [Prerequisites for the virtual device](#prerequisites-for-the-virtual-device).

After completing these procedures, you are ready to [Work with the virtual device](#work-with-the-storsimple-virtual-device).

### Create the virtual device

After you have created a virtual network, configured a StorSimple Manager service, and registered your physical StorSimple device with the service, you can use the following steps to create a StorSimple virtual device.

Perform the following steps to create the StorSimple virtual device.

1.  In the Azure classic portal, go to the **StorSimple Manager** service.

2. Go to the **Devices** page.

3. In the **Create Virtual Device** dialog box, specify the following details.

	![StorSimple create virtual device](./media/storsimple-virtual-device/StorSimple_CreateVirtualDevice1.png)

	1. **Name** – A unique name for your virtual device.

	2. **Virtual Network** – The name of the virtual network that you want to use with this virtual device.

	3. **Subnet** – The subnet on the virtual network for use with the virtual device.

	4. **Storage Account for Virtual Device Creation** – The storage account that will be used to hold the image of the virtual device during provisioning. This storage account should be in the same region as the virtual device and virtual network. It should not be used for data storage by either the physical device or the virtual device. By default, a new storage account will be created for this purpose. However, if you know that you already have a storage account that is suitable for this use, you can select it from the list.

4. Click the check mark to indicate that you understand that the data stored on the virtual device will be hosted in a Microsoft datacenter. A virtual device will now be created. It may take up to 45 minutes to 1 hour to create a virtual device.
	![StorSimple virtual device creating stage](./media/storsimple-virtual-device/StorSimple_VirtualDeviceCreating1M.png)

When you use only a physical device, your encryption key is kept with your device; therefore, Microsoft cannot decrypt it. When you use a virtual device, both the encryption key and the decryption key are stored in Microsoft Azure. For more information, see [Security considerations for using a virtual device](#security-considerations-for-using-a-virtual-device).

### Configure and register the virtual device

Before starting this procedure, make sure that you have a copy of the service data encryption key. The service data encryption key was created when you configured your first StorSimple physical device and you were instructed to save it in a secure location. If you do not have a copy of the service data encryption key, you must [contact Microsoft Support](storsimple-contact-microsoft-support.md) for assistance.

Perform the following steps to configure and register your StorSimple virtual device.


1. Select the **StorSimple virtual device** you just created on the **Devices** page.

2. Click **Complete Device Configuration**. This starts the Configure device wizard.

	![StorSimple complete device setup in Devices page](./media/storsimple-virtual-device/StorSimple_CompleteDeviceSetupSVA1M.png)

1. In the Configure device wizard:

	1. Enter the **Service Data Encryption Key** in the space provided.
	2. Enter the **Snapshot Manager password**. The **Snapshot Manager password** must be 14 or 15 characters in length and must contain a combination of lowercase, uppercase, numeric, and special characters.
	3. Enter the **device administrator password**. The **device administrator password** must be between 8 to 15 characters and must contain a combination of lowercase, uppercase, numeric, and special characters.
	4. Click the check icon to finish the initial configuration and registration of the virtual device.

		![StorSimple virtual device settings](./media/storsimple-virtual-device/StorSimple_VirtualDeviceSettings1.png)

After the configuration and registration is complete, the device will come online. It may take several minutes for the device to come online.

![StorSimple virtual device online stage](./media/storsimple-virtual-device/StorSimple_VirtualDeviceOnline1M.png)

### Modify the device configuration settings

The following section describes the device configuration settings that you may want to configure for the StorSimple virtual device. These settings include those of CHAP, StorSimple Snapshot Manager password, or device administrator password.

#### Configure the CHAP initiator (optional)

This parameter contains the credentials that your virtual device (target) expects from the initiators (servers) that are attempting to access the volumes. The initiators will provide a CHAP user name and a CHAP password to identify themselves to your device during this authentication.

#### Configure the CHAP target (optional)

This parameter contains the credentials that your virtual device uses when a CHAP-enabled initiator requests mutual or bi-directional authentication. Your virtual device will use a Reverse CHAP user name and Reverse CHAP password to identify itself to the initiator during this authentication process. Note that CHAP target settings are global settings. When these are applied, all the volumes connected to the storage virtual device will use CHAP authentication.

#### Configure the StorSimple Snapshot Manager (optional)

StorSimple Snapshot Manager software resides on your Windows host and allows administrators to manage backups of your StorSimple device in the form of local and cloud snapshots.

>[AZURE.NOTE] For the virtual device, your Windows host is an Azure VM.

When configuring a device in the StorSimple Snapshot Manager, you will be prompted to provide the StorSimple device IP address and password to authenticate your storage device. This password is first configured through the Windows PowerShell interface.

Perform the following steps to change StorSimple Snapshot Manager when using it with your StorSimple virtual device.

1. On your virtual device, go to **Devices > Configure**.

- Scroll down to the **Snapshot Manager** section. Enter a password that is 14 or 15 characters. Make sure that the password contains a combination of uppercase, lowercase, numeric, and special characters.

- Confirm the password.

- Click **Save** at the bottom of the page.

The StorSimple Snapshot Manager password is now updated and can be used when you authenticate your Windows hosts.

#### Configure the device administrator password

When you use the Windows PowerShell interface to access the virtual device, you will be required to enter a device administrator password. For the security of your data, you are required to change this password before the virtual device can be used.

Perform the following steps to change the device administrator password for your StorSimple virtual device.

1. On your virtual device, go to **Devices > Configure**.

1. Scroll down to the **Device Administrator password** section. Provide an administrator password that contains from 8 to 15 characters. The password must be a combination of uppercase, lowercase, numeric, and special characters.

1. Confirm the password.

1. Click **Save** at the bottom of the page.

The device administrator password should now be updated. You will use this modified password to access the the Windows PowerShell interface on your virtual device.

#### Configure remote management (optional)

Remote access to your virtual device via the Windows PowerShell interface is not enabled by default. You need to enable remote management on the virtual device first, and then enable it on the client that will be used to access your virtual device.

You can choose to connect over HTTP or HTTPS. For security reasons, we recommend that you use HTTPS with a self-signed certificate to connect to your virtual device.

Perform the following steps to configure remote management for your StorSimple virtual device.

1. On your virtual device, go to **Devices > Configure**.

2. Scroll down to the **Remote Management** section.

3. Set **Enable Remote Management** to **Yes**.

4. You can now choose to connect using HTTP. The default is to connect over HTTPS. Connecting over HTTP is acceptable only on trusted networks.

5. Click **Download Remote Management Certificate** to download a remote management certificate. You will specify a location in which to save this file. This certificate then needs to be installed on the client or host machine that you will use to connect to the virtual device.

6. Click **Save** at the bottom of the page.

![Video available](./media/storsimple-virtual-device/Video_icon.png) **Video available**

To watch a video that describes how to create a virtual StorSimple device in the cloud, click [here](https://azure.microsoft.com/documentation/videos/create-a-storsimple-virtual-device/).

## Work with the StorSimple virtual device

Now that you have created and configured the StorSimple virtual device, you are ready to start working with it. You can work with volume containers, volumes, and backup policies on a virtual device just as you would on a physical StorSimple device; the only difference is that you need to make sure that you select the virtual device from your device list. Refer to the following sections for instructions on associated tasks:


- [Volume containers](storsimple-manage-volume-containers.md)

- [Volumes](storsimple-manage-volumes.md)

- [Backup policies](storsimple-manage-backup-policies.md)

The following sections discuss some of the differences you will encounter when working with the virtual device.

### Maintain a StorSimple virtual device

Because it is a software-only device, maintenance for the virtual device is minimal when compared to maintenance for the physical device. You have the following options:

- **Software updates** – You can view the date that the software was last updated, together with any update status messages. You can use the Scan updates button at the bottom of the page to perform a manual scan if you want to check for new updates.
- **Support package** – You can create and upload a support package to help Microsoft Support troubleshoot issues with your virtual device.

### Storage accounts for a virtual device

Storage accounts are created for use by the StorSimple Manager service, by the virtual device, and by the physical device. When you create your storage accounts, we recommend that you use a region identifier in the friendly name to help ensure that the region is consistent throughout all of the system components. For a virtual device, it is important that all of the components be in the same region to prevent performance issues.

### Deactivate a StorSimple virtual device

Deactivating a virtual device deletes the VM and the resources created when it was provisioned. After the virtual device is deactivated, it cannot be restored to its previous state. Before you deactivate the virtual device, make sure to stop or delete clients and hosts that depend on it.

Deactivating a virtual device results in the following actions:

- The virtual device is removed.

- The OSDisk and Data Disks created for the virtual device are removed.

- The hosted service and virtual network created during provisioning are retained. If you are not using them, you should delete them manually.

- Cloud snapshots created for the virtual device are retained.

As soon as the device is shown as deactivated on the StorSimple Manager service page, you can delete the virtual device from the StorSimple Manager service device list.

### Remotely access a StorSimple virtual device

After you have enabled it on the StorSimple device configuration page, you can use Windows PowerShell remoting to connect to the virtual device from another virtual machine inside the same virtual network; for example, you can connect from the host VM that you configured and used to connect iSCSI. In most deployments, you will have already opened a public endpoint to access your host VM that you can use for accessing the virtual device.

>[AZURE.WARNING] For enhanced security, we strongly recommend that you use HTTPS when connecting to the endpoints and then delete the endpoints after you have completed your PowerShell remote session.

You should follow the procedures in [Connect remotely to your StorSimple device](storsimple-remote-connect.md) to set up remoting for your virtual device.

However, if you want to connect directly to the virtual device from another computer outside the virtual network or outside the Microsoft Azure environment, you need to create additional endpoints as described in the following procedure.

Perform the following steps to create a public endpoint on the virtual device.

1. Sign in to the Azure classic portal.

- Click **Virtual Machines**, and then select the virtual machine that is being used as your virtual device.

- Click **Endpoints**. The Endpoints page lists all endpoints for the virtual machine.

- Click **Add**. The Add Endpoint dialog box appears. Click the arrow to continue.

- For the **Name**, type the following name for the endpoint: **WinRMHttps**.

- For the **Protocol**, specify **TCP**.

- For the **Public Port**, type the port numbers that you want to use for the connection.

- For the **Private Port**, type **5986**.

- Click the check mark to create the endpoint.

After the endpoint is created, you can view its details to determine the Public Virtual IP (VIP) address. Record this address.

We recommend that you connect from another virtual machine inside the same virtual network because this practice minimizes the number of public endpoints on your virtual network. When you use this method, you simply connect to the virtual machine through a Remote Desktop session and then configure that virtual machine for use as you would any other Windows client on a local network. You do not need to append the public port number because the port will already be known.

### Stop and restart

Unlike the physical StorSimple device, there is no power on or power off button to push on a StorSimple virtual device. However, there may be occasions where you need to stop and restart the virtual device. For example, some updates might require that the VM be restarted to finish the update process. The easiest way for you to start, stop, and restart a virtual device is to use the Virtual Machines Management Console.

When you look at the Management Console, the virtual device status is **Running** because it is started by default after it is created. You can stop and restart a virtual machine at any time.

To stop a virtual device, click its name, and then click **Shutdown**. While the virtual device is shutting down, its status is **Stopping**. After the virtual device is stopped, its status is **Stopped**.

When a virtual device is running and you want to restart it, click its name, and then click **Restart**. While the virtual device is restarting, its status is **Restarting**. When the virtual device is ready for you to use, its status is **Running**.

You can also use the following Windows PowerShell cmdlets to start, stop, and restart the virtual device. An example follows each cmdlet.

`Start-AzureVMC:\PS>Start-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`


`Stop-AzureVMC:\PS>Stop-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`

`Restart-AzureVMC:\PS>Restart-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`

### Reset to factory defaults

If you decide that you just want to start over with your virtual device, simply deactivate and delete it and then create a new one. Just like when your physical device is reset, your new virtual device will not have any updates installed; therefore, make sure to check for updates before using it.


## Failover to the virtual device

Disaster recovery (DR) is one of the key scenarios that the StorSimple virtual device was designed for. In this scenario, the physical StorSimple device or entire datacenter might not be available. Fortunately, you can use a virtual device to restore operations in an alternate location. During DR, the volume containers from the source device change ownership and are transferred to the virtual device. The prerequisites for DR are that the virtual device has been created and configured, all the volumes within the volume container have been taken offline, and the volume container has an associated cloud snapshot.

### To restore your physical device to the StorSimple virtual device

1. Verify that the volume container you want to fail over has associated cloud snapshots.

- Open the **Device** page, and then click the **Volume Containers** tab.

- Select a volume container that you would like to fail over to the virtual device. Click the volume container to display the list of volumes within the container. Select a volume and click **Take Offline** to take the volume offline. Repeat this process for all the volumes in the volume container.

- Repeat the previous step for all the volume containers you want to fail over to the virtual device.

- On the **Device** page, select the device that you need to fail over, and then click **Failover** to open the Device Failover wizard.

- In **Choose volume container to failover**, select the volume containers you would like to fail over. To be displayed in this list, the volume container must contain a cloud snapshot and be offline. If a volume container that you expected to see is not present, cancel the wizard and verify that it is offline.

- On the next page, in **Choose a target device for the volumes** in the selected containers, select the virtual device from the drop-down list of available devices. Only the devices that have the available capacity are displayed on the list.

- Review all the failover settings on the **Confirm failover** page. If they are correct, click the check icon.

The failover process will begin. When the failover is finished, go to the **Devices** page and select the virtual device that was used as the target for the failover process. Go to the Volume Containers page. All the volume containers, along with the volumes from the old device should appear.

>[AZURE.NOTE] The amount of storage supported on the virtual device is 30 TB.

![Video available](./media/storsimple-virtual-device/Video_icon.png) **Video available**

To watch a video that describes how you can restore a failed over physical device to a virtual device in the cloud, click [here](https://azure.microsoft.com/documentation/videos/storsimple-and-disaster-recovery/).

## Shut down or delete the virtual device

If you previously configured and used a StorSimple virtual device but now want to stop accruing compute charges for its use, you can shut down the virtual device. Shutting down the virtual device doesn’t delete its operating system or data disks in storage. It does stop charges accruing on your subscription, but storage charges for the OS and data disks will continue.

If you delete or shut down the virtual device, it will appear as **Offline** on the **Devices** page of the StorSimple Manager service. You can choose to deactivate it or delete it as a device if you also wish to delete the backups created by the virtual device. For more information, see [Deactivate and delete a StorSimple device](storsimple-deactivate-and-delete-device.md).

### To shut down the StorSimple virtual device

1. Sign in to the Azure classic portal.

2. Click **Virtual Machines**, and then select the virtual device.

3. Click **Shutdown**.

### To delete the StorSimple virtual device

1. Sign in to the Azure classic portal.

2. Click **Virtual Machines**, and then select the virtual device.

3. Click **Delete** and choose to delete all the virtual machine disks.


## Next steps

To administer your virtual device, refer to the detailed list of workflows in [Administer StorSimple device using StorSimple Manager service](storsimple-manager-service-administration.md#administer-storsimple-device-using-storsimple-manager-service).
