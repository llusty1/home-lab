# Home Network Engineering Lab — Build Documentation

**Lawrence Lusty | Dallas, Texas | April 2026**

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Host System Specifications](#host-system-specifications)
3. [Lab Architecture](#lab-architecture)
4. [Network Design](#network-design)
5. [Virtual Machine Configurations](#virtual-machine-configurations)
6. [Golden Image Strategy](#golden-image-strategy)
7. [Lab Startup Procedure](#lab-startup-procedure)
8. [Connectivity Verification](#connectivity-verification)
9. [Key Troubleshooting Lessons](#key-troubleshooting-lessons)
10. [Next Steps](#next-steps)

---

## Executive Summary

This document details the design, build, and configuration of a professional-grade home network engineering lab built on a Windows workstation using VMware Workstation Pro and GNS3. The lab provides a fully isolated virtual network environment capable of simulating Cisco router topologies, running automated network scripts, and hosting supporting infrastructure including a Kali Linux penetration testing VM and an Ubuntu Server VM.

The lab is designed to serve as a hands-on platform for developing and demonstrating network engineering, automation, and security skills for career development purposes.

---

## Host System Specifications

| Component | Specification |
|---|---|
| Machine Name | COMMAND-CENTER |
| Processor | AMD Ryzen 7 5700G |
| RAM | 64 GB |
| Lab Storage | 1 TB Dedicated (G: Drive) |
| Hypervisor | VMware Workstation Pro |
| Operating System | Windows (Daily Driver) |

---

## Lab Architecture

The lab uses a layered virtualization architecture where VMware Workstation Pro serves as the hypervisor for all virtual machines. GNS3 provides the network simulation layer, running inside its own dedicated VM to maximize stability and performance.

### Architecture Diagram

```
Windows Host (COMMAND-CENTER)
├── VMware Workstation Pro (Hypervisor)
│   ├── GNS3 VM        — Network Simulation Engine  (192.168.216.129)
│   ├── Kali Linux     — Penetration Testing / Python Automation  (192.168.216.130)
│   └── Ubuntu Server  — Infrastructure / GitLab / Services  (192.168.216.131)
└── GNS3 GUI           — Management Interface (runs on Windows host)
```

---

## Network Design

Two VMware virtual networks provide connectivity across the lab:

| Network | Type | Subnet | Purpose |
|---|---|---|---|
| VMnet1 | Host-Only | 192.168.216.0/24 | Lab backplane — all VMs communicate here |
| VMnet8 | NAT | 192.168.74.0/24 | Internet access for VMs |
| VMnet0 | Bridged | 192.168.1.0/24 | Physical network access |

### IP Address Assignments

| Device | VMnet1 IP | Role |
|---|---|---|
| GNS3 VM | 192.168.216.129 | Network simulation engine |
| Kali Linux | 192.168.216.130 | Automation & security testing |
| Ubuntu Server | 192.168.216.131 | Infrastructure services |

---

## Virtual Machine Configurations

### GNS3 VM

| Setting | Value |
|---|---|
| Location | G:\VMware VM's\GNS3 VM\ |
| RAM | 8 GB |
| CPUs | 4 vCPUs |
| Storage | 19.5 GB (OS) + 488.3 GB (Dynamic) |
| Network Adapter 1 | VMnet1 Host-Only — 192.168.216.129 |
| Network Adapter 2 | NAT — Internet Access |
| GNS3 Server Version | 2.2.58.1 |
| KVM Support | Available |

### Kali Linux VM

| Setting | Value |
|---|---|
| Source | Pre-built VMware image (kali-linux-2026.1-vmware-amd64) |
| Location | G:\VMware VM's\kali-linux-2026.1-vmware-amd64\ |
| RAM | 4 GB |
| CPUs | 4 vCPUs |
| Storage | 80.1 GB |
| Network Adapter 1 | Bridged — Internet Access (192.168.1.x) |
| Network Adapter 2 | VMnet1 Host-Only — 192.168.216.130 |
| Snapshot | Golden Image — Clean Install |

### Ubuntu Server VM

| Setting | Value |
|---|---|
| Source | Fresh install from ISO (ubuntu-26.04-live-server-amd64) |
| Location | G:\VMware VM's\Ubuntu Server\ |
| RAM | 4 GB |
| CPUs | 2 vCPUs |
| Storage | 60 GB |
| Install Type | Ubuntu Server (Minimized) |
| Filesystem | ext4 — single partition |
| Network Adapter 1 | Bridged — Internet Access (192.168.1.123) |
| Network Adapter 2 | VMnet1 Host-Only — 192.168.216.131 |
| Snapshot | Golden Image — Clean Install |

---

## Golden Image Strategy

Both Kali Linux and Ubuntu Server have been snapshotted in their clean post-install state before any project-specific software was installed. This follows the professional practice of maintaining golden images for rapid redeployment.

### Golden Image Workflow

1. Install OS and configure base networking
2. Verify connectivity across VMnet1 backplane
3. Take VMware snapshot: **"Golden Image — Clean Install"**
4. Clone VM for project work — never modify the golden image directly
5. Discard clones after project completion or take new snapshots as needed

---

## Lab Startup Procedure

The following sequence must be followed every time the lab is started to ensure proper VM ownership and GNS3 connectivity:

| Step | Action | Notes |
|---|---|---|
| 1 | Open VMware Workstation Pro | Right-click → Run as Administrator |
| 2 | Power on GNS3 VM | Wait for IP screen (192.168.216.129) |
| 3 | Open GNS3 GUI | Right-click → Run as Administrator |
| 4 | Verify Server Summary | Both COMMAND-CENTER and GNS3 VM show green |
| 5 | Power on Kali Linux | Normal power on in VMware |
| 6 | Power on Ubuntu Server | Normal power on in VMware |

> **Important:** Both VMware Workstation Pro and GNS3 GUI must run under the same user context (Administrator) for the GNS3 VM connection to function correctly. This is a VMware process ownership requirement.

---

## Connectivity Verification

| Test | Command | Result |
|---|---|---|
| Kali → GNS3 VM | `ping -c 4 192.168.216.129` | 4/4 packets received |
| Ubuntu → GNS3 VM | `ping -c 4 192.168.216.129` | 4/4 packets received |
| Ubuntu → Kali | `ping -c 4 192.168.216.130` | 4/4 packets received |
| GNS3 API Health | `curl http://192.168.216.129/v2/version` | Version 2.2.58.1 returned |

---

## Key Troubleshooting Lessons

Real issues encountered and resolved during the build. Documented to demonstrate systematic problem solving.

### Issue 1 — GNS3 VM Reverting to Hyper-V Engine

- **Root cause:** GNS3 configuration file `gns3_gui.ini` contained Hyper-V paths from a previous setup
- **Resolution:** Manually edited `gns3_gui.ini` to set correct `vmrun_path` and cleared stale Hyper-V recent file references

### Issue 2 — vmrun Returning Zero Running VMs

- **Root cause:** VMware Workstation GUI held an exclusive file lock on the GNS3 VM `.vmx` file
- **Resolution:** Terminated `vmware-vmx.exe` process to release the lock, allowing vmrun to take ownership

### Issue 3 — GNS3 GUI Loading Blank Configuration When Run as Administrator

- **Root cause:** Launching GNS3 as Administrator caused it to load a different AppData profile than the Lawrence user account
- **Resolution:** Both VMware Workstation and GNS3 GUI must run under the same user context. Running both as Administrator resolved the VM communication issue while preserving the correct configuration

### Issue 4 — VMware preferences.ini Path Mismatch

- **Root cause:** GNS3 VM stored on non-default G: drive with apostrophe in folder name (`VM's`)
- **Resolution:** Corrected `prefvmx.defaultvmpath` in VMware `preferences.ini` to match the exact folder path on the G: drive

---

## Next Steps

| Project | Description | Skills |
|---|---|---|
| Project 1 | Automated Network Inventory Tool | Python, Netmiko, Cisco IOS, GitLab |
| Project 2 | Network Monitoring Dashboard | SNMP, Python, Data Visualization |
| Project 3 | Kubernetes Lab | Container orchestration, DevOps |
| Project 4 | Security Testing Lab | Kali Linux, Penetration Testing |

---

*Home Network Engineering Lab | Lawrence Lusty | April 2026*
