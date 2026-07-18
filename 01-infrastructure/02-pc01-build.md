# Runbook: Building PC01 (Client Workstation)
 
**Date:** July 17-18, 2026
**Author:** Duckboy-mr
 
## Objective
Stand up a domain-joined client workstation (PC01) to test Group Policy application,
simulate end-user workflows, and serve as the test subject for Phase 3 break/fix scenarios.
 
## Prerequisites
- VirtualBox installed
- Windows Server 2022 evaluation ISO (see note below on OS substitution)
- Host-only network configured (192.168.56.0/24)
- DC01 already built and promoted to domain controller for labcorp.local
## Design Decision: OS Substitution
 
The original plan called for Windows 10 as the client OS. I did not have a Windows 10 ISO
on hand and decided not to download one mid-build. Instead, **PC01 runs Windows Server 2022
(Desktop Experience), configured strictly as a domain member server — no server roles
installed, not promoted to a domain controller.**
 
This is functionally equivalent to a Windows 10 client for the purposes of this lab:
- Domain join process is identical (System Properties → Change → Domain).
- Group Policy processes Computer Configuration and User Configuration policies the same
  way regardless of server vs. client SKU.
- Drive mapping, wallpaper, password policy, and USB restriction GPOs all apply identically.
The only real difference is cosmetic (Server Manager launches by default instead of a
consumer Windows shell) and the absence of a genuine "client OS" for portfolio realism.
Documenting this substitution transparently rather than hiding it.
 
## Steps
 
1. **Created the PC01 VM** in VirtualBox — Windows Server 2022 (64-bit) OS type, 2048 MB RAM,
   30 GB dynamically allocated disk, attached to the host-only network adapter.
2. **Installed Windows Server 2022** (Desktop Experience) using the evaluation ISO.
3. **Did not install any server roles** — confirmed PC01 remains a plain member server, not
   a domain controller.
4. **Set a static IP** on PC01 (192.168.56.20), subnet 255.255.255.0, DNS pointed at DC01
   (192.168.56.10) — required for domain join to succeed.
5. **Verified connectivity** to DC01 with `ping 192.168.56.10` before attempting domain join.
6. **Joined PC01 to the labcorp.local domain** via System Properties → Change → Domain.
7. **Renamed the computer to PC01** post-join (see Issues Encountered — this step was
   originally missed).
8. **Logged in as LABCORP\jsmith** to confirm domain authentication worked end-to-end.
## Verification
 
- Confirmed PC01 appears in Active Directory Users and Computers under the Computers
  container with the correct name.
- Confirmed successful domain logon as jsmith.
- Ran `gpresult /r` to confirm applicable GPOs were listed for both computer and user.
## Issues Encountered
 
1. **VM creation — wrong disk selected.** During initial VM setup, "Use an Existing Virtual
   Hard Disk File" was selected by default and pointed at DC01's existing .vdi file instead
   of creating a new one. Fixed by selecting "Create a New Virtual Hard Disk" and confirming
   the path pointed to a new PC01.vdi file.
2. **"Virtual machine path is not unique" error.** A leftover folder from the aborted first
   attempt (`...\VirtualBox VMs\PC01\`) still existed on disk, blocking VM creation under the
   same name. Fixed by deleting the stale folder before recreating the VM.
3. **"No bootable medium found" on first boot.** The Windows Server 2022 ISO was never
   attached to the virtual optical drive — the ISO Image field in the New VM wizard was left
   as `<not selected>`. Fixed via Settings → Storage → attach the ISO manually, or by
   confirming the ISO Image field is actually populated during VM creation next time.
4. **Domain join succeeded but computer kept its auto-generated name** (e.g.
   `WIN-XXXXXXX`) instead of being renamed to PC01 before joining. This meant the AD
   computer object was created under the wrong name. Fixed by renaming the machine via
   Server Manager → Local Server → Computer Name → Change (while already domain-joined,
   which still works and just requires a second credential prompt), then restarting.
5. **Security Filtering on Block-USB-Storage GPO could not find "PC01$".** The
   "Select User, Computer, or Group" dialog's default Object Types only searches Users,
   Groups, and Built-in security principals — Computers is not enabled by default. This
   caused a false "object not found" error even though the PC01 computer account existed
   correctly in AD. Fixed by clicking **Object Types...** in that dialog and checking
   **Computers**, after which `PC01$` resolved normally.
## Lessons Learned
 
Renaming a machine *before* joining it to a domain saves a step — if a rename happens after
domain join, it still works but requires re-entering domain credentials and a second restart.
Also learned that AD object-picker dialogs are scoped by object type per-dialog, not globally,
so "object not found" errors don't always mean the object is missing — they sometimes mean
the search is looking in the wrong category. Worth checking Object Types before assuming
something is broken.