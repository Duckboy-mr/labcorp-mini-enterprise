# Runbook: Shared Folder Setup and NTFS Inheritance Issue
 
**Date:** July 16, 2026
**Author:** Duckboy-mr
 
## Objective
Create department shared folders (Sales, HR, Engineering) on DC01, share them over the
network, and lock down access to only the matching security group for each department —
demonstrating group-based access control instead of per-user permissions.
 
## Steps
 
1. Created `C:\Shares\Sales`, `C:\Shares\HR`, `C:\Shares\Engineering` (initially created in
   the wrong location — see Issues Encountered).
2. Shared each folder (Properties → Sharing → Advanced Sharing → Share this folder), with
   the share name matching the folder name.
3. Set Share permissions: removed the default Everyone entry, added the matching
   department group (e.g. Sales-Users) with Change and Read.
4. Set NTFS permissions (Security tab): added the matching department group with Modify.
## Issues Encountered
 
1. **Folders created in the wrong location.** Initially created under
   `C:\Users\Administrator\Documents\Shares\` instead of the root of C:\. This works
   functionally (shares still work from any path) but is bad practice — system-level
   infrastructure shouldn't live under a personal user profile. Fixed by moving the entire
   Shares folder tree to `C:\Shares\`.
2. **Inherited "Users (LABCORP\Users)" group on NTFS permissions.** After moving the
   folders (and even before, at creation), each folder's NTFS Security tab showed an
   inherited "Users" group with Read/Read & Execute access — a domain-wide group that
   would have let any authenticated domain user read department files, defeating the
   purpose of group-based access control. Windows would not allow removing this entry
   directly because it was inherited from the parent folder (`C:\Shares` or `C:\` itself).
   **Root cause:** NTFS permissions inherit down the folder tree by default. The parent
   `C:\Shares` folder had a broad "Users" group permission (a Windows default), and every
   subfolder created inside it inherited that same broad access automatically.
   **Fix:** On each folder's Security tab → Advanced → **Disable inheritance** →
   **Convert** (keeps existing explicit permissions like Administrators/SYSTEM but breaks
   the link to the parent so they can be edited independently) → then removed the "Users"
   group entry, leaving only the department-specific group with Modify access.
   This had to be repeated individually for Sales, HR, and Engineering, since each folder
   inherits independently.
## Verification
 
- Confirmed via Security tab on Engineering that only SYSTEM, CREATOR OWNER, Administrators,
  and Engineering-Users remain, with Engineering-Users at Modify — no domain-wide group
  present.
- Confirmed the share is reachable and permission-scoped by connecting as jsmith
  (Sales-Users member) to `\\DC01\Sales` and successfully browsing it (see drive-mapping
  runbook for the connectivity test that confirmed this).
## Lessons Learned
 
NTFS permission inheritance is on by default and silently carries broad default groups
(like the domain's built-in "Users" group) down into every subfolder you create, even
inside a folder you set up yourself for a specific purpose. Locking down access properly
means explicitly checking inherited permissions on new folders, not just setting
permissions on the group you intend to grant access to — an oversight here would have
left every department's files readable domain-wide despite looking correctly configured
at a glance.