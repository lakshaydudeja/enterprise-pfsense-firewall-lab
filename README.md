# Enterprise pfSense Firewall Lab

## Project Overview

This project demonstrates the deployment and configuration of a pfSense firewall in a virtualized enterprise-style lab environment. The goal is to secure a small-business-style internal network using firewall rules, NAT, DHCP, DNS, network segmentation, traffic filtering, and security validation.

pfSense is configured as the main firewall and default gateway for the internal lab network. The lab is built using VMware Workstation Pro and includes Windows, Windows Server, Kali Linux, and Splunk systems for testing and validation.

## Lab Objectives

- Deploy pfSense as a virtual firewall
- Configure WAN, LAN, and management interfaces
- Route internal VM traffic through pfSense
- Configure DHCP and DNS services
- Create and test firewall rules
- Configure NAT for outbound internet access
- Validate traffic flow using Windows and Kali Linux
- Review pfSense firewall logs
- Document configuration steps with screenshots

## Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation Pro |
| Firewall | pfSense |
| LAN Network | 10.0.0.0/24 |
| pfSense LAN IP | 10.0.0.1 |
| Windows 10 VM | 10.0.0.2 |
| Windows Server / AD VM | 10.0.0.3 |
| Kali Linux VM | 10.0.0.4 |
| Splunk Server | 10.0.0.5 |
| DMZ Network | 10.0.10.0/24 |
| pfSense DMZ IP | 10.0.10.1 |
| Windows Server 2025 DMZ Server | 10.0.10.10 |
| pfSense Management IP | 172.16.0.2 |

## Network Topology

```text
Internet
   |
VMware NAT / WAN
   |
pfSense Firewall
   |
   |-- LAN Segment: 10.0.0.0/24
   |      |-- Windows 10 Client: 10.0.0.2
   |      |-- Windows Server / Active Directory: 10.0.0.3
   |      |-- Kali Linux: 10.0.0.4
   |      |-- Splunk Server: 10.0.0.5
   |
   |-- DMZ Segment: 10.0.10.0/24
          |-- Windows Server 2025 DMZ Server: 10.0.10.10

Management Access:
Host PC -> 172.16.0.2 -> pfSense Web GUI
