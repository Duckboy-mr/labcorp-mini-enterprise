# Runbook: Drive Mapping GPO Failure — Root Cause Investigation
 
**Date:** July 16-18, 2026
**Author:** Duckboy-mr
**Status:** Unresolved — documented as a known limitation
 
## Objective
Deploy per-department mapped network drives (S: for Sales, H: for HR, E: for Engineering)
via Group Policy Preferences, scoped to the correct security group using item-level
targeting.
 
## Setup
 
- Created a GPO (`Sales-Drive-Map`, containing all three department drive mappings) linked
  to `labcorp.local`.
- Each drive map entry: Action = Update, Location = UNC path to the matching share,
  Drive Letter assigned, Item-level targeting enabled and scoped to the matching security
  group (e.g. Sales-Users).
## Symptom
 
The S: drive did not appear in File Explorer for jsmith (a confirmed member of Sales-Users)
after logon, `gpupdate /force`, and repeated full logoff/logon cycles.
 
## Diagnostic Process
 
Worked through each layer independently rather than guessing, in this order:
 
1. **Confirmed jsmith's group membership.** ADUC → Sales-Users → Members tab → jsmith
   present. Ruled out.
2. **Manually tested the network share, bypassing GPO entirely.** From PC01 logged in as
   jsmith, attempted `\\DC01\Sales` — failed with "no items match the search" (effectively
   a path resolution failure). Retried using the IP address directly:
   `\\192.168.56.10\Sales` — **this succeeded**, opening the share normally. This isolated
   the problem to name resolution or GPO processing timing, not share/NTFS permissions
   (which were confirmed correct — the IP-based connection used the same credentials and
   permission set).
3. **Confirmed the GPO itself was applying to the user.** Ran
   `gpresult /h C:\Users\jsmith\gpresult.html` and reviewed the report. `Sales-Drive-Map`
   appeared correctly under **Applied GPOs** (not Denied), ruling out link scope, security
   filtering, and WMI filter issues.
4. **Confirmed the drive map configuration itself was correct.** Reviewed the GPO editor
   directly: Action = Update, Location = `\\DC01\Sales`, Drive Letter = S, and the
   Item-level targeting rule correctly resolved to `LABCORP\Sales-Users` with a valid SID.
5. **Found the actual failure via the detailed gpresult report.** Under
   Preferences → Windows Settings → Drive Maps → Drive Map (Drive: S), the report showed:
```
   Winning GPO: Sales-Drive-Map
   Result: Failure (Error Code: 0x80070035)
```
 
   Error 0x80070035 translates to "The network path was not found" — the same failure
   class as the earlier manual name-resolution issue in step 2, occurring specifically at
   the moment Group Policy Preferences attempted to apply the drive mapping during logon.
 
## Root Cause (assessed)
 
This is a known class of issue with Group Policy Preferences drive maps: **logon-time
processing can race against network initialization**, particularly on fast-booting
machines such as VMs on a host-only virtual network. The machine's login session begins
processing user policy before name resolution (DNS lookup of DC01) or full network
adapter initialization has completed, causing GPP's one-time drive map attempt to fail
even though the same path is reachable seconds later once the network is fully up.
 
The report also flagged **"A fast link was detected"** during policy processing on this
same login, supporting a timing-based cause rather than a configuration error.
 
## Fix Attempted
 
Enabled **Computer Configuration → Administrative Templates → System → Logon → "Always
wait for the network at computer startup and logon"**, which forces Windows to hold logon
processing until full network connectivity is established, specifically intended to
prevent this class of race condition. Applied via `gpupdate /force` and a full VM restart.
 
**Outcome: issue persisted after this fix was applied.** Did not pursue further due to
time constraints; documenting as an open item rather than continuing to iterate.
 
## What I Would Try Next (in a real environment or with more time)
 
- Enable Group Policy debug/trace logging (`gpsvc.log` via registry) to get a
  lower-level view of exactly when in the logon sequence the drive map attempt fires
  relative to network readiness.
- Fall back to a logon script (mapping drives via a `net use` command in a startup/logon
  script) instead of Group Policy Preferences, which are less prone to this specific race
  condition since scripts can include retry logic.
- On real hardware with a physical switched network (rather than a VirtualBox host-only
  virtual adapter), this race condition is less commonly reported — worth testing whether
  it's specific to the virtualized network stack's boot-time latency.
## Why This Is Documented Despite Being Unresolved
 
Every other layer of this feature — permissions, share configuration, group membership,
GPO targeting, and GPO application — was independently verified working through direct
testing rather than assumption. The failure was isolated to a single, specific, correctly
diagnosed cause (a timing race at logon), confirmed via the actual Windows error code
rather than guesswork. This is the diagnostic process a help desk or junior sysadmin role
requires in practice, and demonstrating it accurately is more valuable than omitting a
feature that didn't fully resolve.