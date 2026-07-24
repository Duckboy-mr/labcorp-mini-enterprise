# Enterprise IT Infrastructure Environment

A fully virtualized enterprise environment demonstrating identity management, network services, and security operations administration.

## Components

| Host | Role | IP Address |
|------|------|------------|
| DC01 | Domain Controller / DNS Server | 192.168.56.10 |
| PC01 | Domain-Joined Client Workstation | 192.168.56.20 |
| SPL01 | Splunk Enterprise SIEM | 192.168.56.30 |

**Network:** VirtualBox Host-Only Network (192.168.56.0/24)

## Documentation

| Category | Contents |
|----------|----------|
| [`01-infrastructure`](01-infrastructure) | Network design, domain controller build, workstation provisioning |
| [`02-policies`](02-policies) | Group Policy deployment, SIEM log forwarding, security configurations |
| [`03-runbooks`](03-runbooks) | Operational procedures, troubleshooting guides, incident response |
| [`04-incidents`](04-incidents) | Documented investigations and root-cause analysis |
| [`05-tickets`](05-tickets) | Sample support tickets and resolution workflows |

## Key Demonstrations

**Identity & Access Management**
- Designed and administered a multi-OU Active Directory domain (`labcorp.local`) with role-based security groups
- Configured and enforced Group Policy Objects for password complexity, USB device restrictions, and drive mappings

**Network Services**
- Deployed DNS and DHCP services on Windows Server 2022
- Managed static IP allocation and name resolution for all infrastructure nodes

**Security Operations**
- Built a Splunk Enterprise SIEM pipeline ingesting Windows Event Logs via Universal Forwarder
- Created detection dashboards visualizing failed authentication attempts (Event ID 4625)

**Troubleshooting & Root Cause Analysis**
- Investigated and isolated a Group Policy drive-mapping failure to a logon-time network race condition
- Resolved NTFS permission inheritance conflicts using systematic layer-by-layer verification (`dcdiag`, `gpresult`, Event Viewer)
