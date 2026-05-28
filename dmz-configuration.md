# DMZ Configuration

## Objective

The objective of this section is to configure a separate DMZ network in the pfSense firewall lab. The DMZ network will host a Windows Server 2025 virtual machine that is separated from the internal LAN.

This setup demonstrates network segmentation by placing server resources in a separate network zone instead of keeping them directly inside the trusted LAN.

## DMZ Network Design

| Component | Configuration |
|---|---|
| DMZ Network Type | VMware LAN Segment |
| DMZ Segment Name | DMZ-Segment |
| DMZ Subnet | 10.0.10.0/24 |
| pfSense DMZ Gateway | 10.0.10.1 |
| Windows Server 2025 DMZ IP | 10.0.10.10 |
| Subnet Mask | 255.255.255.0 |

## Planned DMZ Security Policy

| Traffic Flow | Policy |
|---|---|
| LAN to DMZ | Allow limited administrative access |
| DMZ to LAN | Block by default |
| DMZ to Internet | Allow only required outbound access |
| WAN to DMZ | Allow only published services if configured |
| WAN to LAN | Block |

## Configuration Steps

### Step 1: Create DMZ LAN Segment in VMware

A separate VMware LAN Segment named `DMZ-Segment` will be created for the DMZ network.

This LAN Segment will be connected to:

- pfSense DMZ interface
- Windows Server 2025 DMZ server

### Step 2: Add DMZ Interface to pfSense

A new network adapter will be added to the pfSense virtual machine and connected to the `DMZ-Segment`.

### Step 3: Add Windows Server 2025 to DMZ

The Windows Server 2025 virtual machine will be connected to the same `DMZ-Segment`.

### Step 4: Configure pfSense DMZ Interface

The pfSense DMZ interface will be assigned the IP address:

`10.0.10.1/24`

### Step 5: Configure Windows Server 2025 DMZ IP

The Windows Server 2025 VM will be configured with:

| Setting | Value |
|---|---|
| IP Address | 10.0.10.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.10.1 |
| DNS Server | 10.0.10.1 |

## Completed Configuration

### DMZ Interface Assignment

The DMZ interface was assigned in the pfSense console.

Final interface mapping:

| pfSense Interface | VMware Interface | IP Address |
|---|---|---|
| WAN | em0 | DHCP - 192.168.126.133/24 |
| LAN | em1 | 10.0.0.1/24 |
| HOSTONLY | em2 | 172.16.0.2/24 |
| DMZ | em3 | 10.0.10.1/24 |

### DMZ Interface Configuration

The DMZ interface was enabled and configured with a static IPv4 address.

| Setting | Value |
|---|---|
| Interface Name | DMZ |
| IPv4 Configuration Type | Static IPv4 |
| IPv4 Address | 10.0.10.1 |
| Subnet Mask | /24 |
| Purpose | Gateway for DMZ network |

### Windows Server 2025 DMZ Configuration

Windows Server 2025 was connected to the VMware `DMZ-Segment` LAN segment and configured with a static IP address.

| Setting | Value |
|---|---|
| IP Address | 10.0.10.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.10.1 |
| DNS Server | 10.0.10.1, 8.8.8.8 |

### Temporary DMZ Connectivity Rule

A temporary firewall rule was created on the DMZ interface to allow initial connectivity testing.

| Setting | Value |
|---|---|
| Action | Pass |
| Interface | DMZ |
| Address Family | IPv4 |
| Protocol | Any |
| Source | DMZ subnets |
| Destination | Any |
| Description | TEMP - Allow DMZ traffic for initial connectivity testing |

This temporary rule allowed the DMZ server to reach the pfSense DMZ gateway and access the internet. This rule will later be replaced with more restrictive production-style rules.

## Validation Tests

### Test 1: DMZ Server Internet Access

The Windows Server 2025 DMZ server was tested for outbound internet connectivity.

Command used:

```cmd
ping 8.8.8.8
```

Result:

```text
Successful - 4 packets sent, 4 packets received, 0% packet loss
```

This confirms that the DMZ server can reach the internet through pfSense.

### Test 2: DMZ DNS Resolution

DNS resolution was tested from the Windows Server 2025 DMZ server.

Command used:

```cmd
nslookup google.com
```

Result:

```text
Successful - google.com resolved to public IPv4 and IPv6 addresses
```

This confirms that DNS resolution is working from the DMZ network.

### Test 3: DMZ to LAN Block Test

The Windows Server 2025 DMZ server was tested against the internal LAN gateway.

Command used:

```cmd
ping 10.0.0.1
```

Result:

```text
Blocked - 4 packets sent, 0 packets received, 100% packet loss
```

This confirms that the firewall rule blocking DMZ-to-LAN traffic is working as expected.

### Test 4: LAN to DMZ Admin Access

The Windows 10 LAN client was tested for connectivity to the Windows Server 2025 DMZ server.

Command used:

```cmd
ping 10.0.10.10
```

Result:

```text
Successful - 4 packets sent, 4 packets received, 0% packet loss
```

This confirms that the internal LAN can reach the DMZ server through the firewall rule allowing LAN-to-DMZ administrative access.

## Validation Summary

| Test                    | Source                | Destination | Expected Result | Actual Result |
| ----------------------- | --------------------- | ----------- | --------------- | ------------- |
| Internet connectivity   | DMZ Server            | 8.8.8.8     | Allowed         | Successful    |
| DNS resolution          | DMZ Server            | google.com  | Allowed         | Successful    |
| DMZ to LAN access       | DMZ Server            | 10.0.0.1    | Blocked         | Blocked       |
| LAN to DMZ admin access | Windows 10 LAN Client | 10.0.10.10  | Allowed         | Successful    |

The validation confirms that the DMZ network can access the internet and resolve DNS, access from the DMZ into the trusted LAN is blocked, and the trusted LAN can reach the DMZ server for administrative access.

## Screenshots Collected

| Screenshot                                   | Description                                                                                                           |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `03-pfsense-dmz-interface-ip-configured.png` | Shows the DMZ interface enabled in pfSense with static IPv4 address `10.0.10.1/24`.                                   |
| `06-dmz-firewall-rules-secured.png`          | Shows the final DMZ firewall rules blocking DMZ-to-LAN traffic and allowing DMZ outbound internet access.             |
| `07-dmz-connectivity-validation.png`         | Shows successful internet connectivity, successful DNS resolution, and blocked DMZ-to-LAN access from the DMZ server. |
| `09-lan-to-dmz-ping-success.png`             | Shows successful LAN-to-DMZ connectivity from the Windows 10 LAN client to the Windows Server 2025 DMZ server.        |
