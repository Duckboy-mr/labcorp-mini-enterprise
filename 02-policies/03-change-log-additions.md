# Change Log — Additions (append these rows to 02-policies/03-change-log.md)
 
| Date | Change | Reason | Rolled back? |
|------|--------|--------|---------------|
| 2026-07-16 | Created C:\Shares\Sales, HR, Engineering; moved from initial incorrect location under Documents to C:\Shares | Correct location for infrastructure shares, not under a user profile | No |
| 2026-07-16 | Broke NTFS inheritance and removed inherited "Users" domain group from Sales/HR/Engineering folders | Default inherited permission granted domain-wide read access, defeating group-based access control | No |
| 2026-07-16 | Set Share and NTFS permissions scoped to matching department security group (Change/Read share, Modify NTFS) | Standard group-based access control practice | No |
| 2026-07-16 | Created Password-Policy GPO (12 char min, 5 history) | Meet baseline security requirement | No |
| 2026-07-16 | Created Sales-Drive-Map GPO containing S:/H:/E: drive mappings with item-level targeting per department group | Department file access per role | No — feature non-functional, see 03-runbooks/06-drive-map-troubleshooting.md |
| 2026-07-16 | Created Block-USB-Storage GPO, Computer Configuration policy enabled | DLP requirement | No |
| 2026-07-16 | Removed Authenticated Users from Block-USB-Storage security filtering; scoped to PC01$ computer account instead | Simulate department-level USB exemption in a single-PC lab environment | No |
| 2026-07-17/18 | Built PC01 as Windows Server 2022 domain member (substituted for Windows 10 due to ISO availability) | No Windows 10 ISO on hand; functionally equivalent for AD/GPO demonstration purposes | No |
| 2026-07-17/18 | Renamed PC01 post-domain-join (was left with auto-generated hostname) | Computer account needed to match documented naming convention for security filtering to work as expected | No |
| 2026-07-18 | Enabled "Always wait for the network at computer startup and logon" (Computer Configuration → Administrative Templates → System → Logon) | Attempted fix for drive map GPO failing with error 0x80070035 (network path race condition at logon) | No — did not resolve issue, left enabled as a reasonable general-purpose setting regardless |