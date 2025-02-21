# Creating Multiple Subnets in Azure

## Introduction

Your company wants to block communication between two VMs. They currently reside on the same network and on the same subnet. Ensure they are separated into different subnets and communication is blocked using a network security group.

## Solution

Log in to the Azure Portal using the credentials provided on the lab page.

### Create a New Subnet

1. Click the hamburger menu icon in the upper left (this is the portal menu button).
1. Click **Virtual networks**.
1. Click the provided virtual network.
1. Click **Subnets** in the left-hand menu.
1. Click **+ Subnet**.
1. In the pane that appears, set the following values:
    * *Name*: **subnet2**
    * *Address range (CIDR block)*: **10.0.1.0/24**
1. Click **OK**.

### Create a New Network Security Group

1. Click the portal menu button in the upper left.
1. Click **All Services**.
1. Search for **nsg**.
1. In the search results, click **Network security groups**.
1. Click the provided network security group.
1. Click **Inbound security rules** in the left-hand menu.
1. Click **+ Add**.
1. In the pane that appears, set the following values:
    * *Source*: **IP Addresses**
    * *Source IP addresses/CIDR ranges*: **10.0.1.0/24**
    * *Source port ranges*: *
    * *Destination*: **Any**
    * *Destination port ranges*: *
    * *Protocol*: **Any**
    * *Action*: **Deny**
    * *Priority*: **100**
    * *Name*: **block_all_subnet2**.
1. Click **Add**.
1. Click **Outbound security rules** in the left-hand menu.
1. Click **+ Add**.
1. In the pane that appears, set the following values:
    * *Source*: **IP Addresses**
    * *Source IP addresses/CIDR ranges*: **10.0.0.0/24**
    * *Source port ranges*: *
    * *Destination*: **Any**
    * *Destination IP addresses*: **10.0.1.0/24**
    * *Destination port ranges*: *
    * *Protocol*: **Any**
    * *Action*: **Deny**
    * *Priority*: **100**
    * *Name*: **block_all_subnet2_out**.
1. Click **Add**.

### Move the VM to the New Subnet

 1. Click the portal menu button in the upper left.
 1. Click **Virtual machines**.
 1. Click **linVM**.
 1. Click **Network** in the left-hand menu.
 1. Click the listed network interface (it'll be named something like `nic2-bhfpm`).
 1. Click **IP configurations** in the left-hand menu.
 1. In the *Subnet* dropdown, click to select **subnet2**.
 1. Click **Save**.
 1. Click **Virtual machines** in the breadcrumb link trail at the top of the screen.
 1. Click **winVM**.
 1. Click **Connect** > **RDP**.
 1. Click **Download RDP File**.
 1. Open the RDP file, and log in to the VM using the credentials provided on the lab page.
 1. In the Windows VM, click the Windows menu button in the lower left and search for "CMD".
 1. Select the **Command Prompt** desktop app.
 1. Attempt to ping `subnet2`:

    ```
    ping 10.0.1.4/24
    ```

    The communication is blocked, meaning we were successful. 

## Conclusion

Congratulations — you've completed this hands-on lab!