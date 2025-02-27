<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Set up the domain controller VM in Azure
- Configure the client VM in Azure
- Install and configure Active Directory
- Create domain administrative accounts
- Join the client to the domain
- Configure remote desktop for non-administrative users

<h2>Deployment and Configuration Steps</h2>

## Phase 1: Setting Up Domain Controller in Azure

### Create a Resource Group
1. Log into **Azure Portal** ([portal.azure.com](https://portal.azure.com)).
2. Navigate to **Resource Groups**.
3. Click **"Create"** to create a new resource group.
4. Name it appropriately (e.g., `AD-Lab-RG`).
5. Select your preferred **region**.
6. Click **"Create"**.

### Create a Virtual Network and Subnet
1. Navigate to **Virtual Networks**.
2. Click **"Create"**.
3. Select the **resource group** created in the previous step.
4. Name the VNet (e.g., `AD-Lab-VNet`).
5. Set an appropriate **address space** (e.g., `10.0.0.0/16`).
6. Create a **subnet** (e.g., `AD-Subnet`) with an **address range** (e.g., `10.0.0.0/24`).
7. Click **"Create"**.

### Create the Domain Controller VM
1. Navigate to **Virtual Machines**.
2. Click **"Create" > "Azure Virtual Machine"**.
3. Select the **resource group** and **region**.
4. Name the VM `DC-1`.
5. Select the image **"Windows Server 2022 Datacenter"**.
6. Choose an appropriate size (**at least 2 vCPUs and 4 GB memory**).
7. Set username to **"labuser"**.
8. Set password to **"Cyberlab123!"**.
9. Allow **RDP (3389) inbound port**.
10. Select the **VNet and subnet** created earlier.
11. Click **"Create"**.

### Configure Static IP for Domain Controller
1. Once the VM is created, navigate to **DC-1** in the **Azure Portal**.
2. Go to **"Networking"** under **Settings**.
3. Click on the **network interface**.
4. Go to **"IP configurations"** under **Settings**.
5. Click on the **IP configuration** (usually `ipconfig1`).
6. Change the assignment from **"Dynamic"** to **"Static"**.
7. Click **"Save"**.

### Disable Windows Firewall on DC-1
> **Note:** This is for testing purposes only and not recommended for production environments.

1. Connect to **DC-1** via **Remote Desktop**.
2. Open **Windows Defender Firewall with Advanced Security**.
3. In the left pane, click on **"Windows Defender Firewall Properties"**.
4. Set the **firewall state** to **"Off"** for **Domain, Private, and Public profiles**.
5. Click **"Apply"** and **"OK"**.

---

## Phase 2: Setting Up Client Machine in Azure

### Create the Client VM
1. Navigate to **Virtual Machines**.
2. Click **"Create" > "Azure Virtual Machine"**.
3. Select the **same resource group and region** as `DC-1`.
4. Name the VM `Client-1`.
5. Select the image **"Windows 10 Pro"**.
6. Choose an appropriate **size**.
7. Set username to **"labuser"**.
8. Set password to **"Cyberlab123!"**.
9. Allow **RDP (3389) inbound port**.
10. Select the **same VNet and subnet** as `DC-1`.
11. Click **"Create"**.

### Configure DNS Settings for Client-1
1. Navigate to **Client-1** in the **Azure Portal**.
2. Go to **"Networking"** under **Settings**.
3. Click on the **network interface**.
4. Go to **"DNS servers"** under **Settings**.
5. Select **"Custom"** for DNS servers.
6. Enter **DC-1's private IP address** as the DNS server.
7. Click **"Save"**.

### Restart Client-1
1. In the **Azure Portal**, select `Client-1`.
2. Click **"Restart"** at the top of the overview page.
3. Wait for the VM to restart.

### Test Connectivity
1. Connect to **Client-1** via **Remote Desktop**.
2. Open **Command Prompt**.
3. Ping DC-1's private IP address:

    ```sh
    ping [DC-1-Private-IP]
    ```

4. Verify the **ping is successful**.
5. Open **PowerShell** and run:

    ```sh
    ipconfig /all
    ```

6. Verify that the **DNS server is set to DC-1's private IP address**.

---

## Phase 3: Installing and Configuring Active Directory

### Install Active Directory Domain Services
1. Connect to `DC-1` via **Remote Desktop**.
2. Open **Server Manager** (should open automatically on login).
3. Click **"Add roles and features"**.
4. Click **"Next"** until you reach **"Server Roles"**.
5. Check **"Active Directory Domain Services"**.
6. Click **"Add Features"** when prompted.
7. Click **"Next"** until you reach the confirmation page.
8. Click **"Install"**.
9. Wait for the **installation to complete**.

### Promote Server to Domain Controller
1. In **Server Manager**, click on the **flag icon** with the **yellow warning**.
2. Click **"Promote this server to a domain controller"**.
3. Select **"Add a new forest"**.
4. Enter `"mydomain.com"` as the **Root domain name** (or another name of your choice).
5. Click **"Next"**.
6. Set the **Directory Services Restore Mode password**.
7. Click **"Next"** through the remaining pages, accepting defaults.
8. Review the options and click **"Install"**.
9. The **server will restart automatically**.

### Log in as Domain Administrator
- After restart, log in to `DC-1` as **"mydomain.com\labuser"** with the password `"Cyberlab123!"`.

---

## Phase 4: Creating Domain Administrative Accounts

### Create Organizational Units
1. In **Server Manager**, click **"Tools" > "Active Directory Users and Computers"**.
2. Right-click on the domain (`mydomain.com`).
3. Select **"New" > "Organizational Unit"**.
4. Name it `_EMPLOYEES`.
5. Create another OU named `_ADMINS`.
6. Create a third OU named `_CLIENTS`.

### Create Admin User
1. Right-click on the `_ADMINS` OU.
2. Select **"New" > "User"**.
3. Enter the following:

    - **First name:** Jane  
    - **Last name:** Doe  
    - **User logon name:** jane_admin  

4. Click **"Next"**.
5. Set password to `"Cyberlab123!"`.
6. Uncheck **"User must change password at next logon"**.
7. Check **"Password never expires"**.
8. Click **"Next"**, then **"Finish"**.

### Add User to Domain Admins Group
1. Right-click on **jane_admin**.
2. Select **"Properties"**.
3. Click the **"Member Of"** tab.
4. Click **"Add"**.
5. Type **"Domain Admins"** and click **"Check Names"**.
6. Click **"OK"** twice.

### Log in as New Admin
1. Log out of **DC-1**.
2. Log back in as **"mydomain.com\jane_admin"** with the password `"Cyberlab123!"`.

---

## Phase 5: Joining Client to the Domain
1. Connect to **Client-1** via **Remote Desktop**.
2. Open **System Settings** and click **"Rename this PC (advanced)"**.
3. Click **"Change"**.
4. Select **"Domain"** and enter `"mydomain.com"`.
5. Enter credentials for `"mydomain.com\jane_admin"`.
6. Click **"OK"** to join the domain.
7. Restart the computer when prompted.

---

## Phase 6: Configuring Remote Desktop for Non-Administrative Users
1. Log in to **Client-1** as `"mydomain.com\jane_admin"`.
2. Open **Remote Desktop settings** and allow **"Domain Users"** access.
3. Test Remote Desktop by logging in as any **domain user**.
