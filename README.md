# Zaini SOC Home Lab

![Status](https://img.shields.io/badge/Status-In_Progress-yellow)
![Phase](https://img.shields.io/badge/Current_Phase-1_Complete-brightgreen)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![SIEM](https://img.shields.io/badge/SIEM-Splunk-orange)
![Firewall](https://img.shields.io/badge/Firewall-pfSense-green)
![Attacker](https://img.shields.io/badge/Attacker-Kali_Linux-purple)
![Framework](https://img.shields.io/badge/Framework-MITRE_ATT%26CK-red)
![Certs](https://img.shields.io/badge/Certs-CEH_%7C_CND-darkblue)

A full SOC home lab built on VMware Workstation Pro with pfSense network segmentation, Wazuh SIEM, Splunk, Security Onion IDS, and Shuffle SOAR automation. Designed to simulate enterprise SOC/NOC environments for hands-on skill development, portfolio documentation, and security automation demos.

Built by a senior IT professional with 20+ years of experience in IT operations, cloud infrastructure, and cybersecurity — combining CEH/CND certifications with practical lab work.

-----

## Architecture

```
                         Internet
                            |
                       [VMware NAT]
                            |
                      +-----+-----+
                      |  pfSense  |  ← Central Firewall / Router
                      +-----+-----+
                            |
           +----------------+----------------+
           |                |                |
     [DMZ Subnet]    [AD Subnet]      [SOC Subnet]
     10.10.10.0/24  10.10.20.0/24   10.10.40.0/24
           |                |                |
       Kali Linux      DC01 (AD)        Wazuh SIEM
       DVWA            Win Clients      Splunk
       Metasploitable  zainilab.local   Shuffle SOAR
       Honeypot
           
     [SPAN/Monitor] 10.10.30.0/24
       Security Onion (passive mirror)
```

-----

## Tech Stack

|Category         |Tool                       |Purpose                      |
|-----------------|---------------------------|-----------------------------|
|Hypervisor       |VMware Workstation Pro 25H2|Host platform                |
|Firewall         |pfSense CE 2.7.2           |Network segmentation + NAT   |
|Attacker         |Kali Linux 2026.1          |Offensive testing            |
|Victim Domain    |Windows Server 2022 + AD   |Enterprise simulation        |
|SIEM             |Wazuh 4.14                 |Log analysis + EDR + alerting|
|SIEM             |Splunk Free                |Dashboards + SPL queries     |
|IDS/NSM          |Security Onion             |Network traffic analysis     |
|SOAR             |Shuffle                    |Automated alert response     |
|Vulnerable Target|DVWA + Metasploitable      |Attack practice targets      |
|Threat Intel     |OpenCTI                    |CTI platform (Phase 4)       |

-----

## Phases

### ✅ Phase 1 — Foundation (Week 1)

> VMware configured, pfSense routing 3 isolated subnets, Kali Linux deployed, Windows Server 2022 with Active Directory live

- pfSense CE with 4 interfaces (WAN, LAN/DMZ, AD_SUBNET, SOC_SUBNET)
- Kali Linux 2026.1 on DMZ subnet with internet access via pfSense NAT
- Windows Server 2022 promoted to Domain Controller (zainilab.local)
- AD users and OUs created for attack simulation
- Cross-subnet routing verified

**[→ View Phase 1 Documentation](./phase-1-foundation/README.md)**

-----

### 🔄 Phase 2 — Detection Stack (Week 2)

> Wazuh ingesting logs from AD subnet, Splunk dashboards live, Security Onion capturing traffic

- Wazuh SIEM deployed on SOC subnet
- Wazuh agents installed on DC01 and Kali
- Splunk Free with Universal Forwarder on DC01
- Security Onion on SPAN port (VMnet4)
- First attack scenario: Nmap scan → detected in Wazuh
- Incident report documented with MITRE ATT&CK mapping

**[→ View Phase 2 Documentation](./phase-2-detection/README.md)**

-----

### ⏳ Phase 3 — Automation & SOAR (Week 3)

> Shuffle SOAR automating alert triage, OpenVPN remote access, DVWA attack scenarios

- Shuffle SOAR — Wazuh webhook → auto-response playbooks
- DVWA + Metasploitable deployed on DMZ subnet
- OpenVPN configured for remote lab access
- Honeypot deployed with Wazuh integration
- Firewall rules hardened (permissive → restrictive)

**[→ View Phase 3 Documentation](./phase-3-automation/README.md)**

-----

### ⏳ Phase 4 — Portfolio & Monetisation (Week 4)

> GitHub repo polished, LinkedIn case study published, agency demo delivered

- Full attack scenario library with MITRE ATT&CK mapping
- Complete incident response documentation
- Demo video walkthrough
- Agency client demo via OpenVPN

**[→ View Phase 4 Documentation](./phase-4-portfolio/README.md)**

-----

## Attack Scenarios

|Scenario              |Tools                      |Detection             |Status |
|----------------------|---------------------------|----------------------|-------|
|Network Reconnaissance|Nmap                       |Wazuh + Security Onion|Phase 2|
|AD Enumeration        |BloodHound                 |Wazuh                 |Phase 2|
|Kerberoasting         |Impacket                   |Wazuh + Splunk        |Phase 2|
|Pass-the-Hash         |Mimikatz                   |Wazuh EDR             |Phase 3|
|Web App Attack        |Burp Suite → DVWA          |Security Onion        |Phase 3|
|C2 Simulation         |Metasploit → Metasploitable|Wazuh + Suricata      |Phase 3|

-----

## Repository Structure

```
zaini-soc-homelab/
├── README.md                          ← You are here
├── phase-1-foundation/
│   ├── README.md                      ← Full Phase 1 documentation
│   └── screenshots/                   ← pfSense GUI, AD verification, ping tests
├── phase-2-detection/
│   ├── README.md                      ← Wazuh + Splunk setup + incident report
│   ├── wazuh-rules/                   ← Custom detection rules
│   └── splunk-queries/                ← SPL queries (.spl files)
├── phase-3-automation/
│   ├── README.md                      ← Shuffle SOAR + VPN + hardening
│   └── shuffle-workflows/             ← Exported workflow JSON
└── attack-scenarios/
    └── 01-nmap-reconnaissance/        ← Attack description + evidence + lessons
```

-----

## Key Credentials & IPs (Lab Reference)

|Asset                |IP             |Notes                               |
|---------------------|---------------|------------------------------------|
|pfSense WAN          |192.168.216.128|VMware NAT                          |
|pfSense LAN (DMZ)    |10.10.10.1     |Web GUI at http://10.10.10.1        |
|pfSense AD_SUBNET    |10.10.20.1     |Gateway for AD subnet               |
|pfSense SOC_SUBNET   |10.10.40.1     |Gateway for SOC subnet              |
|Kali Linux           |10.10.10.10    |Attacker machine                    |
|DC01 (Windows Server)|10.10.20.10    |zainilab.local domain controller    |
|Wazuh SIEM           |10.10.40.20    |Dashboard at https://10.10.40.20    |
|Splunk               |10.10.40.30    |Dashboard at http://10.10.40.30:8000|

-----

## About

**Muamad Zaini Bin Rani**  
IT Professional — Singapore  
CEH | CND | Azure | 20+ years IT Operations

This lab is built as both a skill development platform and a portfolio demonstrating hands-on SOC/cybersecurity capability. Every phase is fully documented with real attack traffic, detection evidence, and incident response write-ups.

The Shuffle SOAR integration bridges this lab with my AI automation agency work — demonstrating that security automation and AI-driven workflows are complementary disciplines.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-md--zaini--rani-blue)](https://linkedin.com/in/md-zaini-rani)
[![GitHub](https://img.shields.io/badge/GitHub-za1n177-black)](https://github.com/za1n177)
