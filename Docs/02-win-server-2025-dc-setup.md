# 02 – Windows Server 2025 DC Setup (`project-x-dc`)

This guide walks through building the domain controller, DNS, DHCP, IIS, and basic validation.

## 1. VM Creation

1. Create a new VM:
   - Name: `project-x-dc`
   - OS: Windows Server 2025
   - vCPUs: 2
   - RAM: 4096 MB
   - Disk: 50 GB
2. Attach the Windows Server 2025 ISO.
3. Attach network Adapter 1 to `project-x-nat`.

## 2. Install Windows Server

1. Boot the VM and choose:
   - Install Windows Server
   - Edition: **Desktop Experience (Standard Evaluation)**
2. Accept the EULA and create a new partition on `Disk 0` using defaults.
3. Select the largest partition (e.g., Partition 3) and install.
4. After reboot, set the **Administrator** password:
   - Example: `@Password!`

## 3. Initial Configuration

1. Log in as `Administrator`.
2. Disable screen timeout:
   - **Settings → System → Power → Screen timeout → Never**.
3. Enable CTRL+ALT+DEL in VirtualBox if needed:
   - `Input → Keyboard → Insert Ctrl-Alt-Del`.

## 4. Static IP Configuration

1. Open **Control Panel → Network and Sharing Center → Change adapter settings**.
2. Right-click **Ethernet → Properties → IPv4**.
3. Set a static IP:

   - IP address: `10.0.0.5`
   - Subnet mask: `255.255.255.0`
   - Default gateway: `10.0.0.1`
   - Preferred DNS: (can be left blank for now, will become itself once AD is installed)

4. Click **OK** to save.

## 5. Install Roles and Features

1. Open **Server Manager → Add roles and features**.
2. Use the wizard defaults until the **Server Roles** page.
3. Check:

   - Active Directory Domain Services
   - DHCP Server
   - DNS Server
   - File and Storage Services
   - Web Server (IIS)

4. Proceed with defaults and click **Install**.
5. When finished, use the notification to **Promote this server to a domain controller**.

## 6. Promote to Domain Controller

1. Choose **Add a new forest**.
2. Root domain: `corp.local`.
3. Set Directory Services Restore Mode (DSRM) password (you can reuse `@Password!` for the lab).
4. Accept defaults:
   - NetBIOS: `CORP`
   - Create DNS delegation: leave unchecked.
5. Run the prerequisite checks, then click **Install**.
6. Let the server reboot.

After reboot, log in as **CORP\Administrator**.

## 7. DNS Forwarder

1. In **Server Manager**, open **DNS → DNS Manager**.
2. Right-click the server → **Properties → Forwarders**.
3. Click **Edit** and add:

   - `8.8.8.8` (Google DNS)

4. Apply and close.

Validate:

powershell

ping google.com
nslookup corp.project-x-dc.com

## 8. DHCP Scope

Open DHCP Manager (Server Manager → Tools → DHCP).

Expand IPv4 → New Scope.

Name the scope: project-x-scope.

Configure:

Start IP: 10.0.0.100

End IP: 10.0.0.200

Subnet mask: 255.255.255.0

Router (Gateway): 10.0.0.1

Accept other defaults. Activate the scope.

## 9. Create Lab Users

Open Active Directory Users and Computers.

Navigate to CORP.local → Users.

Create users like johnd and janed:

Set password: @Password123!

Optionally check “User cannot change password” for lab stability.

## 10. Validation

From the DC:

Confirm domain controllers list:

Get-ADDomainController


Confirm DNS and DHCP services are running via Server Manager and MMC consoles.
