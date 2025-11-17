```markdown
# 05 – Corp Server, Docker & MailHog (`project-x-corp-svr`)

(Combines “STEP 4” + “STEP 5” and ties into MailHog configs & scripts.)

## 1. Convert Linux Client into Corp Server

For simplicity, you can re-purpose `project-x-linux-client` into `project-x-corp-svr` or build a separate Ubuntu VM. Below assumes Ubuntu Desktop/Server at `10.0.0.8`.

### 1.1 Change IP Address

1. Open **Settings → Network → Wired**.
2. Edit IPv4 settings:

   - Address: `10.0.0.8`
   - Netmask: `255.255.255.0`
   - Gateway: `10.0.0.1`
   - DNS: `10.0.0.5`

3. Apply and reconnect.

### 1.2 Change Hostname

```bash
sudo hostnamectl set-hostname corp-svr
Log out and back in if needed.

2. Create project-x-admin Account
bash
Copy code
sudo adduser project-x-admin
# password: @Password123!
sudo usermod -aG sudo project-x-admin
Log out the old user and log in as project-x-admin.

3. Install Docker Engine
Follow the official Docker instructions for Ubuntu (summarized):

bash
Copy code
sudo apt update
sudo apt install ca-certificates curl gnupg
# Add Docker's GPG key and repo (check docs.docker.com for the latest commands)
# Then:
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
Test Docker:

bash
Copy code
sudo docker run hello-world

## 4. Deploy MailHog with Docker Compose
From the repo perspective, the compose file lives at configs/docker/mailhog-docker-compose.yml.

On the VM:

bash
Copy code
cd /home/project-x-admin
mkdir mailhog
cd mailhog
sudo nano docker-compose.yml
Paste Code:

version: "3"
services:
  mailhog:
    image: mailhog/mailhog
    container_name: mailhog
    ports:
      - "1025:1025"
      - "8025:8025"

Exit Nano with CTRL + X, then Y key, Enter Key.

Run MailHog:
bash
Copy code
sudo docker compose up -d
MailHog web UI: http://localhost:8025 (inside the VM).

## 5. Send a Test Email from Python
Use scripts/mailhog/test_message.py on the corp server or another Linux host; see that file for details.
Navigate back to the terminal under the /home directory. Create a new file titled sudo nano test_message.py.
Paste in the following code for this test message:

import smtplib
from email.message import EmailMessage

msg = EmailMessage()
msg.set_content("This is a test email from Ubuntu VM.")
msg["Subject"] = "Hello World from MailHog!"
msg["From"] = "corpserver@example.com"
msg["To"] = "user@example.com"

with smtplib.SMTP("localhost", 1025) as server:
    server.send_message(msg)

Exit Nano with CTRL + X, then Y key, Enter Key.

Make sure this python script has executable permissions with: 
sudo chmod +x test_message.py.

Run this python script with: 
sudo python3 test_message.py.


## 6. Email Poller Script on Linux Client
From the Linux client at 10.0.0.101, copy scripts/mailhog/email_poller.sh, install curl + jq, and run it to automatically show new mail to janed.

Let's download a few packages. curl will be used to fetch our email server URL. jq is used to filter JSON strings. 
  sudo apt install curl jq
  cd /home

Create a new Bash script with Nano:
  sudo nano email_poller.sh

Add the following Bash script.

#!/bin/bash

MAILHOG_IP="10.0.0.8"  
TO_EMAIL="janed"
POLL_INTERVAL=30  # seconds

echo "📡 Janed's Mail Watcher started... polling every $POLL_INTERVAL seconds"
echo "🔎 Watching for new mail sent to: $TO_EMAIL@"

# Keep track of seen message IDs
SEEN_IDS_FILE="/tmp/mailhog_seen_ids_janed.txt"
touch "$SEEN_IDS_FILE"

while true; do
  # Fetch current message list
  curl -s http://$MAILHOG_IP:8025/api/v2/messages | jq -c '.items[]' | while read -r msg; do
    TO=$(echo "$msg" | jq -r '.To[].Mailbox')
    ID=$(echo "$msg" | jq -r '.ID')

    if [[ "$TO" == "$TO_EMAIL" && ! $(grep -Fx "$ID" "$SEEN_IDS_FILE") ]]; then
      SUBJECT=$(echo "$msg" | jq -r '.Content.Headers.Subject[0]')
      BODY=$(echo "$msg" | jq -r '.Content.Body')

      echo -e "\n📬 New Email Received!"
      echo "Subject: $SUBJECT"
      echo "From: $(echo "$msg" | jq -r '.Content.Headers.From[0]')"
      echo "Date: $(echo "$msg" | jq -r '.Created')"
      echo -e "Message:\n$BODY"
      echo "-----------------------------------"

      echo "$ID" >> "$SEEN_IDS_FILE"
    fi
  done

  sleep "$POLL_INTERVAL"
Done

Exit Nano with CTRL + X, then Y key, Enter Key.

-	Make email_poller.sh script exectuable:
chmod +x email_poller.sh

-	We can run this script in the background with:
sudo ./email_poller.sh &

-	Stop the script if needed with:
pkill -f email_poller
