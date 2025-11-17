```markdown
# 08 – Sample Lab Scenarios

Use these to demonstrate hands-on experience on your resume and in interviews.

## Scenario 1 – Account Compromise & Login Monitoring

**Goal:** Detect suspicious logins to `johnd` from multiple systems.

Steps:

1. Log into `project-x-win-client` as `CORP\johnd`.
2. From Kali (`project-x-attacker`), attempt RDP or SMB connection with invalid credentials to the Win client.
3. Observe failed login events in:
   - Windows Event Viewer (Security logs).
   - Wazuh dashboard (search for user `johnd` and event ID 4625).

Deliverable:

- Short write-up + screenshots: “Detecting failed logon attempts with Wazuh from domain client”.

## Scenario 2 – Phishing Simulation with MailHog

**Goal:** Simulate a phishing email and show how it is delivered internally.

1. Ensure `corp-svr` MailHog container is running.
2. Run `test_message.py` from `scripts/mailhog/` to send an email.
3. On the Linux client, run `email_poller.sh`.
4. See the new email printed to the terminal for `janed`.

Deliverable:

- Explain how you built a contained “internal email system”.
- Show how you can script polling and alerting.

## Scenario 3 – Linux Domain Join and Access Control

**Goal:** Use AD credentials to access resources from Linux.

1. Log into `project-x-linux-client` as `CORP+janed`.
2. Access a file share or internal web application hosted on the DC or corp server.
3. Use `id`, `wbinfo -u`, and `getent passwd` to demonstrate domain integration.

Deliverable:

- Short document: “Integrating Ubuntu 22.04 with Active Directory using Winbind and Kerberos.”

## Scenario 4 – Security Onion Packet Analysis (Optional Extension)

If you mirror traffic to Security Onion:

1. Generate traffic (web browsing, SMB, RDP) from clients.
2. Use Security Onion’s web interface (e.g., Kibana / TheHive depending on profile) to inspect alerts.
3. Document a small network-based detection (e.g., suspicious HTTP request).

---

You can add many more scenarios: ransomware simulation (in a safe VM), brute force attacks from Kali, privilege escalation paths, etc.
4. Config Files
configs/docker/mailhog-docker-compose.yml
yaml
Copy code
version: "3"
services:
  mailhog:
    image: mailhog/mailhog
    container_name: mailhog
    ports:
      - "1025:1025"
      - "8025:8025"
configs/samba/smb.conf.example
ini
Copy code
[global]
   kerberos method = secrets and keytab
   realm = CORP.PROJECT-X-DC.COM
   workgroup = CORP
   security = ads
   template shell = /bin/bash
   winbind enum groups = Yes
   winbind enum users = Yes
   winbind separator = +
   idmap config * : rangesize = 1000000
   idmap config * : range = 1000000-19999999
   idmap config * : backend = autorid
configs/wazuh/notes-agent-config.md
markdown
Copy code
# Wazuh Agent Config Notes

- Wazuh server IP (example): `10.0.0.10`
- Windows agent name: `project-x-win-client`
- Registration key: generated via `/var/ossec/bin/manage_agents` on the server.
- Service control on Windows:

```powershell
NET START WAZUH
NET STOP WAZUH
Keep this file updated with your actual IPs and keys (do not commit secrets).

python
Copy code

---

## 5. Scripts

### `scripts/mailhog/test_message.py`

```python
import smtplib
from email.message import EmailMessage

SMTP_HOST = "localhost"
SMTP_PORT = 1025

msg = EmailMessage()
msg.set_content("This is a test email from Ubuntu VM.")
msg["Subject"] = "Hello World from MailHog!"
msg["From"] = "corpserver@example.com"
msg["To"] = "janed@example.com"

if __name__ == "__main__":
    with smtplib.SMTP(SMTP_HOST, SMTP_PORT) as server:
        server.send_message(msg)
    print("Test email sent via MailHog.")
scripts/mailhog/email_poller.sh
bash
Copy code
#!/bin/bash

MAILHOG_IP="10.0.0.8"
TO_EMAIL="janed"
POLL_INTERVAL=30  # seconds

echo "📡 Janed's Mail Watcher started... polling every $POLL_INTERVAL seconds"
echo "🔎 Watching for new mail sent to: $TO_EMAIL@"

SEEN_IDS_FILE="/tmp/mailhog_seen_ids_janed.txt"
touch "$SEEN_IDS_FILE"

while true; do
  curl -s "http://${MAILHOG_IP}:8025/api/v2/messages" | jq -c '.items[]' | while read -r msg; do
    TO=$(echo "$msg" | jq -r '.To[].Mailbox')
    ID=$(echo "$msg" | jq -r '.ID')

    if [[ "$TO" == "$TO_EMAIL" && ! $(grep -Fx "$ID" "$SEEN_IDS_FILE") ]]; then
      SUBJECT=$(echo "$msg" | jq -r '.Content.Headers.Subject[0]')
      BODY=$(echo "$msg" | jq -r '.Content.Body')
      FROM=$(echo "$msg" | jq -r '.Content.Headers.From[0]')
      CREATED=$(echo "$msg" | jq -r '.Created')

      echo
      echo "📬 New Email Received!"
      echo "Subject: $SUBJECT"
      echo "From:    $FROM"
      echo "Date:    $CREATED"
      echo "Message:"
      echo "$BODY"
      echo "-----------------------------------"

      echo "$ID" >> "$SEEN_IDS_FILE"
    fi
  done

  sleep "$POLL_INTERVAL"
done
Requires curl and jq:

bash
Copy code
sudo apt install curl jq
scripts/helpers/validate-connectivity.md
markdown
Copy code
# Validate Connectivity – Quick Commands

Run these from different hosts to prove the lab is wired correctly.

## From `project-x-dc` (Windows Server)

```powershell
ping 10.0.0.100      # Win client
ping 10.0.0.101      # Linux client
ping 10.0.0.8        # Corp server
ping google.com      # Internet via DNS forwarder
nslookup corp.local
Get-ADDomainController
From project-x-win-client
powershell
Copy code
ping 10.0.0.5        # DC
ping corp.local
whoami               # should show CORP\johnd
From project-x-linux-client
bash
Copy code
ping -c4 10.0.0.5
ping -c4 corp.local
wbinfo -u            # list domain users
id                   # show CORP+janed context when logged in as domain user
From corp-svr (MailHog host)
bash
Copy code
sudo docker ps       # should show mailhog container
curl http://localhost:8025
