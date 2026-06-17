# Network Topology

## Objective

This section documents the network topology used in the enterprise pfSense firewall lab.

The lab is designed to simulate a small enterprise network with separate security zones for WAN, LAN, DMZ, and management access. pfSense is used as the central firewall and router between these networks.

## Topology Overview

The lab uses VMware Workstation Pro with pfSense acting as the main firewall.

pfSense separates the environment into four network zones:

| Zone | Subnet / Addressing | Purpose |
|---|---|
| WAN | 192.168.126.0/24 | Simulated external network through VMware NAT |
| LAN | 10.0.0.0/24 | Trusted internal network |
| DMZ | 10.0.10.0/24 | Server network for public-facing services |
| HOSTONLY / Management | 172.16.0.0/24 | Host-only management network for pfSense GUI access |

## Network Diagram

```text
Host Computer
     |
     | Management Access
     |
172.16.0.0/24 HOSTONLY Network
     |
pfSense Management Interface
172.16.0.2
     |
     |
Internet / VMware NAT
192.168.126.0/24
     |
pfSense WAN Interface
192.168.126.133
     |
+-----------------------------+
|        pfSense Firewall     |
|                             |
| WAN      em0                |
| LAN      em1                |
| HOSTONLY em2                |
| DMZ      em3                |
+-----------------------------+
     |
     |-----------------------------|
     |                             |
LAN Network                  DMZ Network
10.0.0.0/24                 10.0.10.0/24
     |                             |
pfSense LAN Gateway          pfSense DMZ Gateway
10.0.0.1                     10.0.10.1
     |                             |
Windows 10 Client             Windows Server 2025
10.0.0.2                      10.0.10.10
                               IIS Web Server
```

## pfSense Interface Mapping

| pfSense Interface | VMware Interface | IP Address         | Zone       |
| ----------------- | ---------------- | ------------------ | ---------- |
| WAN               | em0              | 192.168.126.133/24 | WAN        |
| LAN               | em1              | 10.0.0.1/24        | LAN        |
| HOSTONLY          | em2              | 172.16.0.2/24      | Management |
| DMZ               | em3              | 10.0.10.1/24       | DMZ        |

## Virtual Machines

| Virtual Machine | Network Zone | IP Address | Purpose |
|---|---|---|
| pfSense | WAN / LAN / DMZ / Management | Multiple interfaces | Main firewall and router |
| Windows 10 Client | LAN | 10.0.0.2 | Internal trusted workstation |
| Windows Server 2025 | DMZ | 10.0.10.10 | IIS web server hosted in DMZ |

## Network Zone Details

### WAN Zone

The WAN zone simulates external network access using VMware NAT networking.

| Setting        | Value                              |
| -------------- | ---------------------------------- |
| Network Type   | VMware NAT                         |
| Subnet         | 192.168.126.0/24                   |
| pfSense WAN IP | 192.168.126.133                    |
| Purpose        | Simulated external/WAN-side access |

The WAN network was used to test access to the DMZ web server through pfSense NAT port forwarding.

WAN-side test URL:

```text
http://192.168.126.133:8085
```

### LAN Zone

The LAN zone represents the trusted internal network.

| Setting             | Value                    |
| ------------------- | ------------------------ |
| Network Type        | VMware LAN Segment       |
| Subnet              | 10.0.0.0/24              |
| pfSense LAN Gateway | 10.0.0.1                 |
| Windows 10 Client   | 10.0.0.2                 |
| Purpose             | Trusted internal network |

The Windows 10 LAN client was used to test internal access to the DMZ web server.

LAN-to-DMZ test URL:

```text
http://10.0.10.10
```

### DMZ Zone

The DMZ zone hosts the Windows Server 2025 IIS web server.

| Setting                    | Value              |
| -------------------------- | ------------------ |
| Network Type               | VMware LAN Segment |
| DMZ Segment Name           | DMZ-Segment        |
| Subnet                     | 10.0.10.0/24       |
| pfSense DMZ Gateway        | 10.0.10.1          |
| Windows Server 2025 DMZ IP | 10.0.10.10         |
| Hosted Service             | IIS Web Server     |

The DMZ is separated from the trusted LAN using pfSense firewall rules.

DMZ systems are allowed outbound internet access, but DMZ-to-LAN access is blocked.

### Management Zone

The management zone provides host-only access to the pfSense web interface.

| Setting               | Value                             |
| --------------------- | --------------------------------- |
| Network Type          | VMware Host-only                  |
| Subnet                | 172.16.0.0/24                     |
| pfSense Management IP | 172.16.0.2                        |
| Purpose               | pfSense web GUI management access |

The management interface allows the host computer to manage pfSense without relying on the LAN or WAN interfaces.

Management URL:

```text
https://172.16.0.2
```

## Traffic Flow Summary

| Traffic Flow               | Result              | Purpose                                           |
| -------------------------- | ------------------- | ------------------------------------------------- |
| LAN to Internet            | Allowed             | Internal client outbound access                   |
| DMZ to Internet            | Allowed             | DMZ server outbound access and DNS resolution     |
| DMZ to LAN                 | Blocked             | Protect trusted LAN from DMZ systems              |
| LAN to DMZ                 | Allowed             | Allow trusted LAN to access/administer DMZ server |
| WAN to DMZ Web Server      | Allowed on TCP 8085 | Publish DMZ IIS web server through NAT            |
| WAN to LAN                 | Blocked             | Prevent external access to trusted LAN            |
| Host to pfSense Management | Allowed             | Firewall administration                           |

## NAT Traffic Flow

The DMZ IIS web server is published to the WAN side using a NAT port forward.

```text
WAN-side client
     |
     | HTTP request to pfSense WAN IP on TCP 8085
     |
pfSense WAN Interface
192.168.126.133:8085
     |
     | NAT Port Forward
     |
DMZ Web Server
10.0.10.10:80
```

Port mapping:

| External Access     | Internal Destination |
| ------------------- | -------------------- |
| pfSense WAN IP:8085 | 10.0.10.10:80        |

## Security Design

The topology follows a basic enterprise segmentation model:

* The LAN is treated as the trusted internal network.
* The DMZ is used for server services that may need controlled access from other networks.
* The WAN represents an external or untrusted network.
* The management network is separated from normal user and server traffic.
* DMZ-to-LAN traffic is blocked to protect trusted internal systems.
* WAN-to-DMZ access is limited to the published IIS service on TCP port `8085`.
* WAN-to-LAN access remains blocked.

## Validation Performed

| Test                      | Source            | Destination         | Result     |
| ------------------------- | ----------------- | ------------------- | ---------- |
| DMZ internet connectivity | DMZ Server        | 8.8.8.8             | Successful |
| DMZ DNS resolution        | DMZ Server        | google.com          | Successful |
| DMZ to LAN block          | DMZ Server        | 10.0.0.1            | Blocked    |
| LAN to DMZ web access     | Windows 10 Client | 10.0.10.10:80       | Successful |
| WAN to DMZ web access     | WAN-side Client   | pfSense WAN IP:8085 | Successful |

## Related Documentation

| File                     | Purpose                                                             |
| ------------------------ | ------------------------------------------------------------------- |
| `dmz-configuration.md`   | Documents DMZ setup, IIS configuration, validation, and screenshots |
| `firewall-rules.md`      | Documents WAN, LAN, DMZ firewall rules and security policy          |
| `nat-port-forwarding.md` | Documents WAN TCP 8085 to DMZ web server port forwarding            |
