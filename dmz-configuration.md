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

## Validation Tests

The following tests will be performed after configuration:

- Confirm Windows Server 2025 can ping pfSense DMZ gateway
- Confirm LAN can reach DMZ only on allowed ports
- Confirm DMZ cannot access LAN by default
- Confirm DMZ server can access internet if allowed
- Confirm firewall logs show allowed and blocked traffic

## Screenshot Placeholders

Screenshots will be added after the lab configuration is completed.

- pfSense DMZ interface assignment
- pfSense DMZ IP configuration
- Windows Server 2025 static IP configuration
- DMZ firewall rules
- Successful LAN to DMZ test
- Blocked DMZ to LAN test
- Firewall log validation
