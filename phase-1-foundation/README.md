# Phase 1 — Foundation & Network Setup

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Phase](https://img.shields.io/badge/Phase-1%20of%204-blue)
![Duration](https://img.shields.io/badge/Duration-Week%201-orange)

## Overview

Phase 1 establishes the core network infrastructure for the SOC home lab. This includes a segmented virtual network with pfSense as the central firewall/router, an attacker machine (Kali Linux) on the DMZ subnet, and a victim Windows Server 2022 domain controller running Active Directory.

By the end of this phase, three isolated subnets are routing through pfSense with NAT to the internet, and a realistic enterprise Active Directory domain (`zainilab.local`) is live with test users ready for attack simulations in Phase 2.

-----

## Network Architecture

```
                    Internet
                       |
                  [VMware NAT]
                  192.168.216.2
                       |
                 +-----+-----+
                 |  pfSense  |  192.168.216.128 (WAN)
                 |  Firewall |
                 +-----+-----+
                       |
          +------------+------------+
          |            |            |
    [VMnet2]       [VMnet3]     [VMnet5]
  DMZ/Attacker    AD Subnet    SOC Subnet
  10.10.10.0/24  10.10.20.0/24 10.10.40.0/24
       |               |            |
   [Kali Linux]   [DC01 WinSrv]  [Wazuh - Week 2]
   10.10.10.10    10.10.20.10    [Splunk - Week 2]

  [VMnet4] — SPAN/Monitor 10.10.30.0/24
  Reserved for Security Onion (Week 3)
```

-----

## Virtual Machine Inventory

|VM Name          |OS                 |IP Address                            |VMnet           |RAM|CPU|Role              |
|-----------------|-------------------|--------------------------------------|----------------|---|---|------------------|
|pfSense-FW       |pfSense CE 2.7.2   |WAN: 192.168.216.128 / LAN: 10.10.10.1|NAT + VMnet2/3/5|2GB|2  |Firewall / Router |
|kali-linux-2026.1|Kali Linux 2026.1  |10.10.10.10                           |VMnet2          |4GB|2  |Attacker / Pentest|
|DC01-WinSrv2022  |Windows Server 2022|10.10.20.10                           |VMnet3          |4GB|2  |Domain Controller |

-----

## VMware Network Configuration

|VMnet |Type     |Subnet          |Purpose              |
|------|---------|----------------|---------------------|
|VMnet0|Bridged  |—               |External (unused)    |
|VMnet2|Host-only|10.10.10.0/24   |DMZ / Attacker subnet|
|VMnet3|Host-only|10.10.20.0/24   |AD / Victim subnet   |
|VMnet4|Host-only|10.10.30.0/24   |SPAN monitor (Week 3)|
|VMnet5|Host-only|10.10.40.0/24   |SOC subnet           |
|VMnet8|NAT      |192.168.216.0/24|WAN / Internet access|

-----

## pfSense Interface Configuration

|Interface|Name      |Physical|IP Address        |Purpose              |
|---------|----------|--------|------------------|---------------------|
|WAN      |wan       |em0     |192.168.216.128/24|Internet (VMware NAT)|
|LAN      |lan       |em1     |10.10.10.1/24     |DMZ / Attacker subnet|
|OPT1     |AD_SUBNET |em2     |10.10.20.1/24     |Windows AD subnet    |
|OPT2     |SOC_SUBNET|em3     |10.10.40.1/24     |SOC tools subnet     |

**Gateway:** 192.168.216.2 (VMware NAT gateway)

**Firewall Rules:** Pass Any/Any on LAN, AD_SUBNET, SOC_SUBNET (permissive for build phase — will be hardened in Phase 3)

**Outbound NAT:** Automatic mode — translates all internal subnets to WAN address

-----

## Active Directory Configuration

**Domain:** `zainilab.local`  
**Domain Controller:** `DC01.zainilab.local`  
**Forest/Domain Functional Level:** Windows Server 2016  
**DNS:** DC01 (127.0.0.1 self-hosted)

### Organisational Units

|OU             |Path                                   |Purpose                          |
|---------------|---------------------------------------|---------------------------------|
|IT             |OU=IT,DC=zainilab,DC=local             |IT department users              |
|Finance        |OU=Finance,DC=zainilab,DC=local        |Finance department users         |
|ServiceAccounts|OU=ServiceAccounts,DC=zainilab,DC=local|Service accounts (attack targets)|

### Test Users

|Name     |Username  |OU             |Password    |Notes                                           |
|---------|----------|---------------|------------|------------------------------------------------|
|Alice Tan|alice.tan |Finance        |Password123!|Standard user                                   |
|Bob Lim  |bob.lim   |IT             |Password123!|IT user                                         |
|SvcBackup|svc.backup|ServiceAccounts|Password123!|Service account — Kerberoasting target in Week 2|


> ⚠️ **Note:** Weak passwords are intentional for attack simulation purposes. This domain is isolated and not connected to any production network.

-----

## Build Steps Completed

### Day 1 — VMware & Network

- [x] Downloaded pfSense CE 2.7.2, Kali Linux 2026.1, Windows Server 2022 Eval, Wazuh OVA
- [x] Installed VMware Workstation Pro 25H2
- [x] Created VMnets 2, 3, 4, 5, 8 with correct subnets and DHCP disabled
- [x] Created pfSense VM (FreeBSD 64-bit, 20GB, 2GB RAM, 4 NICs)
- [x] Installed pfSense 2.7.2 via ZFS stripe
- [x] Assigned WAN (em0), LAN (em1), OPT1 (em2), OPT2 (em3)
- [x] Configured pfSense web GUI via Setup Wizard
- [x] Enabled OPT1 (AD_SUBNET) and OPT2 (SOC_SUBNET) interfaces
- [x] Added Pass Any firewall rules on all interfaces
- [x] Configured VMnet8 NAT and WANGW gateway (192.168.216.2)
- [x] Verified pfSense WAN internet access (ping 8.8.8.8 from pfSense)
- [x] Imported Kali Linux 2026.1 VMware image
- [x] Upgraded Kali VM hardware compatibility to Workstation 25H2
- [x] Assigned Kali to VMnet2, set static IP 10.10.10.10
- [x] Verified Kali → pfSense connectivity (ping 10.10.10.1)
- [x] Verified Kali → internet via pfSense NAT (ping 8.8.8.8)

### Day 2 — Windows Server & Active Directory

- [x] Created Windows Server 2022 VM (60GB, 4GB RAM, VMnet3)
- [x] Installed Windows Server 2022 Standard (Desktop Experience)
- [x] Set static IP 10.10.20.10, gateway 10.10.20.1, DNS 127.0.0.1
- [x] Renamed computer to DC01
- [x] Installed Active Directory Domain Services role
- [x] Promoted DC01 to Domain Controller (new forest: zainilab.local)
- [x] Created OUs: IT, Finance, ServiceAccounts
- [x] Created users: Alice Tan, Bob Lim, SvcBackup
- [x] Verified with `Get-ADDomain` and `Get-ADUser`
- [x] Took VMware snapshots: Phase1-Clean-Baseline on all 3 VMs

-----

## Verification Tests

### Kali → pfSense (intra-subnet)

```bash
ping 10.10.10.1 -c 3
# Result: 3 packets transmitted, 3 received, 0% packet loss
```

### Kali → Internet (NAT routing)

```bash
ping 8.8.8.8 -c 3
# Result: 3 packets transmitted, 3 received, 0% packet loss
```

### Active Directory verification

```powershell
Get-ADDomain
# DNSRoot: zainilab.local
# PDCEmulator: DC01.zainilab.local

Get-ADUser -Filter * | Select Name, Enabled
# Alice Tan   : True
# Bob Lim     : True
# SvcBackup   : True
```

-----

## Issues Encountered & Resolved

|Issue                      |Root Cause                                |Resolution                                                        |
|---------------------------|------------------------------------------|------------------------------------------------------------------|
|pfSense web GUI unreachable|Windows host had no IP on VMnet2          |Set static IP 10.10.10.100 on VMware Network Adapter VMnet2       |
|WAN showed “no carrier”    |VMnet8 (NAT network) did not exist        |Created VMnet8 as NAT type in Virtual Network Editor              |
|NAT rules not generating   |No default gateway registered in pfSense  |Added WANGW gateway (192.168.216.2) in System → Routing → Gateways|
|Kali unable to resolve DNS |/etc/resolv.conf empty                    |Added nameserver 8.8.8.8 to /etc/resolv.conf                      |
|Kali VM failed to start    |Hardware compatibility was Workstation 8.x|Upgraded VM compatibility to Workstation 25H2                     |

-----

## Lessons Learned

- VMware Workstation Pro 25H2 requires VMnet8 to be explicitly created for NAT to work — it is not created automatically on fresh installs
- pfSense automatic NAT rules only generate when a default gateway is properly registered via System → Routing → Gateways
- Bridged networking on WiFi adapters does not work reliably for VM WAN connections — NAT via VMnet8 is the correct approach for laptop-based labs
- Windows Server AD promotion requires DNS pointing to 127.0.0.1 (itself) before running the wizard

-----

## Next Phase

**[Phase 2 — Detection Stack →](../phase-2-detection/README.md)**

- Deploy Wazuh SIEM on SOC subnet (10.10.40.20)
- Install Wazuh agents on DC01 and Kali
- Deploy Splunk Free with Universal Forwarder on DC01
- Deploy Security Onion on VMnet4 SPAN port
- Run first attack scenario: Nmap scan from Kali → detect in Wazuh
- Document incident report with MITRE ATT&CK mapping

-----

*Built by Muamad Zaini Bin Rani — CEH | CND | Azure | 20+ years IT Operations*  
*[LinkedIn](https://linkedin.com/in/md-zaini-rani) | [GitHub](https://github.com/za1n177)*
