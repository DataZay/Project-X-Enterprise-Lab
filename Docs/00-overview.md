# 00 – Lab Overview

This document provides the high-level overview of the Cyber Academy Project-X lab.

## 1. Purpose

The goal of this environment is to simulate a small enterprise network where you can:

- Practice Windows Server and Active Directory administration.
- Configure Windows and Linux clients in a domain.
- Deploy and interact with Dockerized services (MailHog).
- Use Security Onion and Wazuh for security monitoring.
- Run realistic cyber lab scenarios (phishing, lateral movement, detection).

## 2. Core Components

1. **Directory Services**
   - Windows Server 2025 domain controller.
   - Domain: `corp.local`
   - DNS, DHCP, and IIS web services.

2. **Workstations**
   - Windows 11 Enterprise client (`project-x-win-client`).
   - Ubuntu Desktop client (`project-x-linux-client`) joined to AD.

3. **Infrastructure Servers**
   - Ubuntu based `project-x-corp-svr` (10.0.0.8) running Docker and MailHog.
   - Optional Ubuntu `project-x-sec-box` (10.0.0.10) for additional tools.

4. **Security Stack**
   - Security Onion workstation (`project-x-sec-work`).
   - Wazuh SIEM server (on Ubuntu) + Windows agent.

## 3. Build Order

Follow the guides in this order:

1. **01-lab-architecture.md**
2. **02-win-server-2025-dc-setup.md**
3. **03-win11-enterprise-client-setup.md**
4. **04-ubuntu-linux-client-ad-join.md**
5. **05-corp-svr-docker-mailhog.md**
6. **06-security-onion-setup.md**
7. **07-wazuh-setup-and-agent.md**
8. **08-sample-lab-scenarios.md** – practice exercises.
