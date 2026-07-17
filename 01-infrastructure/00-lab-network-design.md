# Lab Network Design

## Topology
- All 3 VMs connect via VirtualBox Host-Only Network (192.168.56.0/24)
- DC01 acts as DNS server for the network

## IP Plan
| Host  | Role              | IP             |
|-------|-------------------|----------------|
| DC01  | Domain Controller | 192.168.56.10  |
| PC01  | Client Workstation| 192.168.56.20  |
| SPL01 | Splunk/Log Server | 192.168.56.30  |

## Why host-only networking
Host-only keeps the lab isolated from my home network for safety while still letting the VMs
communicate with each other and with my host machine.