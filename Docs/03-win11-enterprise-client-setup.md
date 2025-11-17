# (Windows 11 install + bypass Microsoft account + static IP + join domain.)

# 03 – Windows 11 Enterprise Client Setup (`project-x-win-client`)

## 1. VM Creation

- Name: `project-x-win-client`
- OS: Windows 11 Enterprise
- vCPUs: 2
- RAM: 4096 MB
- Disk: 80 GB
- Network: Adapter 1 → `project-x-nat`

## 2. Install Windows 11

1. Boot from the Windows 11 ISO.
2. Follow the wizard:
   - Install Windows 11 64-bit.
   - Accept license.
   - Create a partition on `Disk 0` and install to the largest partition.

## 3. Bypass Microsoft Account (Local Account Creation)

When prompted to sign in with a Microsoft Account:

1. In VirtualBox, temporarily change network from `NAT Network` to **Host-only Adapter** to cut Internet.
2. Back in the VM, press `Shift + F10` to open a command prompt.
3. Run:

   ```cmd
   oobe\bypassnro
The VM restarts and offers “I don’t have Internet”.

Choose a local username and password (this will later be replaced by domain login).

Disable all privacy toggles for less telemetry.

After setup completes, shut down the VM and switch the network adapter back to NAT Network → project-x-nat.

## 4. Set Static IP and DNS
Ensure project-x-dc is running.

Open Control Panel → Network and Sharing Center → Change adapter settings.

Right-click Ethernet → Properties → IPv4.

Configure:

IP address: 10.0.0.100

Subnet mask: 255.255.255.0

Default gateway: 10.0.0.1

Preferred DNS server: 10.0.0.5 (Domain Controller)

Save settings.

## 5. Join the Domain
In the search bar, type “Change workgroup name” and open System Properties.

Click Change… under Computer Name.

Set:

Computer name: project-x-win-client

Member of: Domain → corp.project-x-dc.com or corp.local (depending on how you prefer FQDN vs AD domain here; corp.local is typical).

When prompted, supply domain credentials for johnd:

Username: johnd@corp.local

Password: @Password123!

You should see a Welcome to the CORP domain message.

Reboot when prompted.

## 6. Log In as a Domain User
At the login screen, choose Other user.

Log in as:

Username: CORP\johnd or johnd@corp.local

Password: @Password123!

Verify the machine appears under Computers in Active Directory Users and Computers on the DC.

This workstation is now ready for Wazuh agent installation and user-based scenarios.
