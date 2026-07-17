# Runbook: Building DC01 (Domain Controller)

**Date:** July 17, 2026
**Author:** [Duckboy-mr]

## Objective
Stand up a Windows Server 2022 domain controller for the labcorp.local domain.

## Prerequisites
- VirtualBox installed
- Windows Server 2022 evaluation ISO downloaded
- Host-only network configured (192.168.56.0/24)

## Steps

1. **Created the DC01 VM** in VirtualBox — Windows Server 2022, 2048 MB RAM, 40 GB dynamically
   allocated disk, attached to the host-only network adapter.

2. **Installed Windows Server 2022** (Desktop Experience) using the evaluation ISO.

3. **Set a static IP** on DC01 (192.168.56.10) so it could reliably act as the network's DNS server.

4. **Installed the Active Directory Domain Services (AD DS) role** through Server Manager.

5. **Promoted DC01 to a domain controller**, creating a new forest with the domain name
   `labcorp.local`.

6. **Created Organizational Units** (Sales, HR, Engineering, IT, Service Accounts) and the
   5 users (jsmith, mgarcia, dlee, admin, svc_backup) in their respective OUs.

## Verification

Ran `dcdiag` from Command Prompt on DC01 to confirm the domain controller is healthy —
all tests passed with no errors.

![dcdiag output](../screenshots/dc01-dcdiag-output.png)

Confirmed all 5 users appear correctly in Active Directory Users and Computers under their
assigned OUs.

![ADUC users list](../screenshots/dc01-aduc-users.png)

## Issues Encountered

[I had an issue with Ctrl+Alt+Del with my laptop keyboard, by issue I mean the Del key didnt even exist but I fixed it using VirtualBox's Input menu.]

## Lessons Learned

[I think a domain controller is what handles logins and keeps track of users on a network. OUs are like folders that help organize everything, and a static IP is useful because the DNS server needs a permanent address that other computers can always find.]