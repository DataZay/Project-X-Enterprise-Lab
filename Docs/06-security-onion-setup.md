# 06 – Security Onion Setup (`project-x-sec-work`)

## 1. Install Security Onion

1. Create VM:
   - Name: `project-x-sec-work`
   - OS: Security Onion 2.4.x ISO
   - vCPU: 1
   - RAM: 2048 MB (or more)
   - Disk: 55 GB
   - Network: Adapter 1 → `project-x-nat`
2. Boot from the ISO and select **Install Security Onion Desktop**.
3. Follow the prompts and create local account:
   - Username: `project-x-sec-work`
   - Password: `@Password123!`

## 2. Network Settings

During setup:

- Hostname: `project-x-sec-work`
- IPv4: `10.0.0.103/24`
- Default Gateway: `10.0.0.1`
- DNS: defaults + `corp.project-x-dc.com` (if prompted)

Accept defaults where appropriate; answer **Yes** to prompts you want for the desktop analyst experience.

## 3. Post-Install

1. Log in as `project-x-sec-work`.
2. Become root and set root password:

```bash
sudo passwd root
# set to @Password123! for lab
From here, you can integrate Security Onion with SPAN / port-mirroring or use it for log review if forwarders are configured.

Later, you can expand this doc with your exact Security Onion profile and sensors.
