# pfSense Setup

## Objective

This section documents the initial pfSense firewall setup used in the enterprise firewall lab.

pfSense was deployed as a virtual firewall in VMware Workstation Pro and configured with separate interfaces for WAN, LAN, DMZ, and management access.

## Lab Platform

| Component       | Details                                         |
| --------------- | ----------------------------------------------- |
| Hypervisor      | VMware Workstation Pro                          |
| Firewall OS     | pfSense                                         |
| Deployment Type | Virtual Machine                                 |
| Purpose         | Firewall, router, NAT, and network segmentation |

## pfSense Role in the Lab

pfSense acts as the central firewall and router between multiple network zones.

| Zone                  | Purpose                                         |
| --------------------- | ----------------------------------------------- |
| WAN                   | Simulated external network through VMware NAT   |
| LAN                   | Trusted internal network                        |
| DMZ                   | Server network for public-facing services       |
| HOSTONLY / Management | Host-only access for pfSense web GUI management |

## VMware Network Adapter Layout

| pfSense Adapter   | pfSense Interface | Network Type       | Purpose                       |
| ----------------- | ----------------- | ------------------ | ----------------------------- |
| Network Adapter 1 | WAN / em0         | VMware NAT         | Simulated external/WAN access |
| Network Adapter 2 | LAN / em1         | VMware LAN Segment | Trusted internal LAN          |
| Network Adapter 3 | HOSTONLY / em2    | VMware Host-only   | Management access             |
| Network Adapter 4 | DMZ / em3         | VMware LAN Segment | DMZ server network            |

## Interface Configuration

| pfSense Interface | Interface Name | IP Address         | Purpose                    |
| ----------------- | -------------- | ------------------ | -------------------------- |
| WAN               | em0            | 192.168.126.133/24 | External/WAN-side access   |
| LAN               | em1            | 10.0.0.1/24        | Gateway for trusted LAN    |
| HOSTONLY          | em2            | 172.16.0.2/24      | pfSense web GUI management |
| DMZ               | em3            | 10.0.10.1/24       | Gateway for DMZ network    |

## Initial Interface Assignment

The pfSense console was used to assign network interfaces.

Final interface assignment:

```text
WAN      -> em0
LAN      -> em1
HOSTONLY -> em2
DMZ      -> em3
```

## Management Access

The pfSense web interface is accessed from the host computer using the HOSTONLY management interface.

Management URL:

```text
https://172.16.0.2
```

This allows firewall administration without relying on the LAN, DMZ, or WAN networks.

## WAN Configuration Note

The WAN interface receives an IP address from VMware NAT.

Current lab WAN IP:

```text
192.168.126.133
```

Because VMware NAT uses private RFC1918 addressing, the following pfSense WAN setting was disabled for lab testing:

```text
Block private networks and loopback addresses
```

This was required only because the lab WAN is private. In a real public-facing deployment, this setting is usually kept enabled unless private WAN addressing is intentionally used.

## LAN Configuration

The LAN interface is used as the default gateway for trusted internal systems.

| Setting           | Value       |
| ----------------- | ----------- |
| LAN Subnet        | 10.0.0.0/24 |
| pfSense LAN IP    | 10.0.0.1    |
| Windows 10 Client | 10.0.0.2    |

## DMZ Configuration

The DMZ interface is used as the default gateway for the DMZ server network.

| Setting                    | Value          |
| -------------------------- | -------------- |
| DMZ Subnet                 | 10.0.10.0/24   |
| pfSense DMZ IP             | 10.0.10.1      |
| Windows Server 2025 DMZ IP | 10.0.10.10     |
| Hosted Service             | IIS Web Server |

## Setup Outcome

The pfSense firewall was successfully configured with separate interfaces for WAN, LAN, DMZ, and management access.

This setup provides the foundation for:

* Network segmentation
* DMZ server hosting
* LAN-to-DMZ controlled access
* DMZ-to-LAN blocking
* WAN-to-DMZ NAT port forwarding
* Centralized firewall rule management

## Related Documentation

| File                     | Purpose                                             |
| ------------------------ | --------------------------------------------------- |
| `network-topology.md`    | Documents the full lab topology and security zones  |
| `dmz-configuration.md`   | Documents DMZ setup, IIS web server, and validation |
| `firewall-rules.md`      | Documents firewall rules and traffic policy         |
| `nat-port-forwarding.md` | Documents WAN-to-DMZ NAT forwarding                 |
