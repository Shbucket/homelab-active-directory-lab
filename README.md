# Active Directory Homelab

## Overview
Built a simulated enterprise Active Directory environment on Proxmox VE,
including a domain controller, 1,000+ provisioned users, organizational
unit structure, Group Policy, DHCP, file sharing, remote access, and
audit logging.

## Environment
- **Hypervisor:** Proxmox VE on Dell OptiPlex 7060
- **Domain Controller:** Windows Server 2022 (DC01, corp.local)
- **Client:** Windows 10 Pro (Win10Client), domain-joined
- **Users:** 1,000+ provisioned via PowerShell automation
- **OU Structure:** IT, HR, Finance, Management, Workstations, Servers

## What Was Built

### Active Directory
- Promoted Windows Server 2022 to a domain controller (corp.local)
- Provisioned 1,000+ users via PowerShell, organized into OUs
- PowerShell script for bulk user creation: [`scripts/create-ad-users.ps1`](scripts/create-ad-users.ps1)
- Created security groups (e.g., IT-Staff) and assigned users
- Joined Windows 10 client to the domain

### Group Policy
- **Default Domain Policy** — password complexity, minimum length (10
  characters), account lockout policy (5 attempts / 30 min lockout)
- **Desktop Restrictions Policy** — blocked Control Panel/Settings access
  for standard users via Administrative Templates, with Domain Admins
  excluded from the policy via security group filtering
- **Drive Mapping Policy** — automatically maps a network share (Z:) to
  all domain users at logon via Group Policy Preferences

### DHCP
- Migrated DHCP services from the home router to the domain controller
- Configured scope (10.0.0.50–10.0.0.200) with gateway and DNS options

### File Sharing
- Created a shared folder (CompanyShare) with appropriate share and NTFS
  permissions for Domain Users

### Remote Access
- Enabled Remote Desktop on the Windows 10 client
- Added a standard domain user to the Remote Desktop Users group
  (least-privilege access rather than granting full admin rights)

### Audit Logging
- Configured Advanced Audit Policy (logon events, account lockout,
  credential validation) via Group Policy
- Verified logon/authentication events appearing in Event Viewer Security
  log (e.g., Event ID 4648)

## Lessons Learned / Troubleshooting

### Xfinity IPv6 DNS Override
Domain join initially failed with "the domain could not be contacted"
despite correct IPv4 DNS configuration pointing to the domain controller.
Root cause: the ISP-provided IPv6 DNS servers (Xfinity) were being
prioritized over IPv4 DNS, and those IPv6 servers had no knowledge of
corp.local. Fixed by removing the IPv6 DNS entries on both the domain
controller and client.

### GPO Security Filtering and Admin Lockout
The Desktop Restrictions Policy initially applied to all authenticated
users, including Administrator — blocking access to Settings (including
the Remote Desktop toggle) for everyone. Fixed by adding Domain Admins to
the GPO's security filtering and removing Authenticated Users, so
administrators retain full access while standard users are restricted.

### Security Log Access Requires Elevated Permissions
Standard domain users cannot view the Security event log by default
(by design). Verifying audit policy required either an administrator
account or adding the user to the Event Log Readers group.

## Next Steps
- Microsoft 365 developer tenant integration with Entra ID and Intune
- Network monitoring (LibreNMS/Zabbix) via SNMP and syslog
- Capstone network operations dashboard (FastAPI + React)
