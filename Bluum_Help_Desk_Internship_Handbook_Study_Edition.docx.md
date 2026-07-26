# Bluum Help Desk Internship Handbook (Study Edition)

This study handbook is an independently created training guide based on common IT help desk practices. It is not an internal Bluum document or based on confidential materials.

## Section 1 — Windows Basics

Windows 11 Settings

Device Manager

Services

Event Viewer

Control Panel

Command Prompt

PowerShell basics

Practice: Find computer name, IP address, rename PC, create/remove local account.

## Section 2 — Password Reset SOP

Verify user identity.

Open Microsoft Entra Admin Center.

Locate the user account.

Reset password.

Require password change at next sign-in.

Unlock account if necessary.

Confirm successful login.

Common causes: Caps Lock, expired password, disabled account, locked account, MFA issue.

## Section 3 — Microsoft Entra ID

Create, disable, delete, and restore users.

Assign licenses.

Manage groups.

Reset MFA.

Review sign-in logs.

Lab: Create test users and practice account lifecycle tasks.

## Section 4 — Microsoft 365

Outlook: mailbox, activation, shared mailboxes.

Teams: microphone, camera, meeting issues.

OneDrive sync troubleshooting.

Office activation and licensing.

## Section 5 — Printers

Install network printers using IP address.

Install drivers.

Print test page.

Troubleshoot offline printers, paper jams, queues, drivers, print spooler.

Useful command: services.msc → Restart Print Spooler.

## Section 6 — Wi-Fi Troubleshooting

Determine if issue affects one user or many.

Forget and reconnect to Wi-Fi.

Restart device.

Commands:

ipconfig /release

ipconfig /renew

ipconfig /flushdns

ping google.com

## Section 7 — Switch Basics

Managed vs unmanaged switches.

PoE.

Access vs trunk ports.

VLAN basics.

Verify link lights, cables, VLAN assignment, and port status.

## Section 8 — Router Basics

Gateway, DHCP, DNS, NAT, LAN/WAN.

Useful commands: ipconfig, ping, tracert, nslookup, arp -a, netstat.

## Section 9 — Ticket Documentation

Document the problem, actions taken, resolution, and time spent.

Example: Teacher unable to print → Verified connectivity → Restarted spooler → Cleared queue → Test page successful.

## Section 10 — Active Directory Concepts

Organizational Units

Security Groups

Group Policy

Users

Computers

Understand hybrid AD + Microsoft Entra environments.

## Section 11 — School Device Deployment

Asset tag device.

Enroll in Intune/MDM.

Join Microsoft Entra.

Install Microsoft 365.

Apply updates.

Deliver to user.

## Section 12 — Common School Tickets

Password reset

Printer offline

Chromebook Wi-Fi

Projector issue

Teams camera

Slow Internet

## Section 13 — Troubleshooting Mindset

Ask: What changed? When did it begin? One user or everyone? Error message? What has already been tried?

## Section 14 — Keyboard Shortcuts

Win+R

Win+E

Win+X

Win+I

Ctrl+Shift+Esc

Alt+Tab

Ctrl+C/V

## Section 15 — PowerShell Basics

hostname

whoami

Get-Service

Get-Process

Get-ComputerInfo

Test-NetConnection

## Recommended Practice Labs

Reset passwords

Install printers

Troubleshoot Wi-Fi

Configure switch

Join device to Entra ID

Deploy Windows laptop

Document help desk tickets

## Suggested Public References

- Microsoft Learn (Entra ID, Microsoft 365)
- Windows printer support documentation
- HP, Xerox, Ricoh, Canon printer manuals
- Cisco Networking Academy (network fundamentals)
- Microsoft Windows networking documentation

---

*Created: 7/26/2026, 5:48:16 PM*
*Document ID: 54da41c4-5c59-461d-b12d-468a26bd56d7*