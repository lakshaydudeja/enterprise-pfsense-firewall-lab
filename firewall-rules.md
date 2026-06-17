# Firewall Rules and NAT Policy

## Objective

This section documents the firewall rules and NAT policies configured in the enterprise pfSense firewall lab.

The goal is to demonstrate how pfSense can be used to enforce network segmentation between the trusted LAN, DMZ, WAN, and management networks.

## Network Zones

| Zone                  | Subnet / Address   | Purpose                                     |
| --------------------- | ------------------ | ------------------------------------------- |
| WAN                   | 192.168.126.133/24 | External/WAN-side access through VMware NAT |
| LAN                   | 10.0.0.0/24        | Trusted internal network                    |
| DMZ                   | 10.0.10.0/24       | Server network hosting the DMZ web server   |
| HOSTONLY / Management | 172.16.0.0/24      | Host-only management access to pfSense      |

## Key Hosts

| Host | Zone | IP Address | Purpose |
|---|---|---|
| pfSense LAN Gateway | LAN | 10.0.0.1 | Default gateway for trusted LAN |
| Windows 10 Client | LAN | 10.0.0.2 | Internal client used for validation |
| Windows Server 2025 DMZ | DMZ | 10.0.10.10 | IIS web server hosted in DMZ |
| pfSense DMZ Gateway | DMZ | 10.0.10.1 | Default gateway for DMZ |
| pfSense Management Interface | HOSTONLY | 172.16.0.2 | Web GUI management access |

## Firewall Policy Overview

The firewall policy follows a basic enterprise-style segmentation model:

| Traffic Flow          | Policy                                            |
| --------------------- | ------------------------------------------------- |
| WAN to LAN            | Block                                             |
| WAN to DMZ            | Allow only TCP 8085 to DMZ web server through NAT |
| LAN to Internet       | Allow                                             |
| LAN to DMZ            | Allow administrative access to DMZ server         |
| DMZ to LAN            | Block                                             |
| DMZ to Internet       | Allow outbound internet access                    |
| Management to pfSense | Allow web GUI access                              |

## WAN Firewall Rules

The WAN interface is used to simulate external access through the VMware NAT network.

A WAN rule was created automatically by pfSense when the NAT port forward was configured.

| Action | Protocol | Source | Destination | Port | Purpose                                                      |
| ------ | -------- | ------ | ----------- | ---- | ------------------------------------------------------------ |
| Pass   | TCP      | Any    | WAN address | 8085 | Allow WAN-side access to DMZ web server through port forward |

### WAN Rule Purpose

This rule allows external/WAN-side clients to connect to the pfSense WAN IP on TCP port `8085`.

Traffic is then forwarded to the IIS web server hosted in the DMZ.

Traffic flow:

```text
WAN Client -> pfSense WAN IP:8085 -> DMZ Web Server 10.0.10.10:80
```

## LAN Firewall Rules

The LAN interface represents the trusted internal network.

| Action | Protocol | Source      | Destination | Purpose                                       |
| ------ | -------- | ----------- | ----------- | --------------------------------------------- |
| Pass   | Any      | LAN subnets | 10.0.10.10  | Allow LAN administrative access to DMZ server |
| Pass   | Any      | LAN subnets | Any         | Default LAN outbound access                   |

### LAN Rule Purpose

The LAN-to-DMZ rule allows the trusted Windows 10 LAN client to access the Windows Server 2025 DMZ server.

This was validated by accessing the IIS web server from the LAN client using:

```text
http://10.0.10.10
```

## DMZ Firewall Rules

The DMZ interface hosts the Windows Server 2025 IIS web server.

The DMZ rules are designed to allow outbound internet access while blocking access into the trusted LAN.

| Rule Order | Action | Protocol | Source      | Destination | Purpose                                            |
| ---------- | ------ | -------- | ----------- | ----------- | -------------------------------------------------- |
| 1          | Block  | Any      | DMZ subnets | LAN subnets | Prevent DMZ systems from accessing the trusted LAN |
| 2          | Pass   | Any      | DMZ subnets | Any         | Allow DMZ outbound internet access                 |

### DMZ Rule Purpose

The first rule blocks DMZ-to-LAN traffic. This protects the trusted LAN if the DMZ server is compromised.

The second rule allows the DMZ server to reach the internet for basic connectivity, updates, and DNS resolution.

Rule order is important. The block rule is placed above the allow rule so that DMZ-to-LAN traffic is denied before the broader outbound allow rule is evaluated.

## NAT Port Forwarding

A NAT port forward was configured to publish the DMZ web server to the WAN side.

| Setting              | Value                          |
| -------------------- | ------------------------------ |
| Interface            | WAN                            |
| Protocol             | TCP                            |
| External Port        | 8085                           |
| Destination          | WAN address                    |
| Redirect Target IP   | 10.0.10.10                     |
| Redirect Target Port | 80                             |
| Description          | WAN TCP 8085 to DMZ Web Server |

### NAT Purpose

The NAT rule forwards traffic from:

```text
pfSense WAN IP:8085
```

to:

```text
10.0.10.10:80
```

This allows WAN-side access to the IIS web server without exposing the service directly on external port `80`.

## WAN Private Network Configuration Note

Because this lab uses VMware NAT networking, the pfSense WAN interface received a private RFC1918 IP address:

```text
192.168.126.133
```

To allow WAN-side testing from the VMware private network, the following pfSense WAN setting was disabled:

```text
Block private networks and loopback addresses
```

This was done only for the lab environment. In a real internet-facing deployment, this setting is usually kept enabled unless the upstream network design specifically requires private WAN addressing.

## Validation Results

| Test                  | Source                | Destination         | Expected Result | Actual Result |
| --------------------- | --------------------- | ------------------- | --------------- | ------------- |
| DMZ internet access   | DMZ Server            | 8.8.8.8             | Allowed         | Successful    |
| DMZ DNS resolution    | DMZ Server            | google.com          | Allowed         | Successful    |
| DMZ to LAN access     | DMZ Server            | 10.0.0.1            | Blocked         | Blocked       |
| LAN to DMZ access     | Windows 10 LAN Client | 10.0.10.10          | Allowed         | Successful    |
| LAN to DMZ web access | Windows 10 LAN Client | 10.0.10.10:80       | Allowed         | Successful    |
| WAN to DMZ web access | WAN-side client       | pfSense WAN IP:8085 | Allowed         | Successful    |

## Security Outcome

The configured firewall and NAT rules successfully enforce the intended segmentation model:

* The DMZ server can access the internet.
* The DMZ server cannot initiate access into the trusted LAN.
* The trusted LAN can access the DMZ server.
* WAN-side clients can access only the published web service through TCP port `8085`.
* WAN-to-LAN access remains blocked.
* The DMZ server is isolated from internal trusted systems while still providing controlled web service access.

## Screenshots

Screenshots for the firewall and NAT configuration are included in the `screenshots/dmz/` folder.

| Screenshot                             | Description                                                                                               |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `06-dmz-firewall-rules-secured.png`    | Shows the final DMZ firewall rules blocking DMZ-to-LAN traffic and allowing DMZ outbound internet access. |
| `12-wan-8085-to-dmz-web-nat-rule.png`  | Shows the NAT port forward from WAN TCP `8085` to DMZ server TCP `80`.                                    |
| `14-wan-firewall-rule-8085-to-dmz.png` | Shows the WAN firewall rule allowing TCP `8085` traffic.                                                  |
| `15-dmz-to-lan-block-validation.png`   | Shows DMZ-to-LAN traffic being blocked.                                                                   |

