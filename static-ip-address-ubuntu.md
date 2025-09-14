# Ubuntu Static IP Configuration Guide

This guide explains how to configure a static IP address in Ubuntu using Netplan.

## Prerequisites

- Ubuntu system with Netplan (Ubuntu 17.10+)
- Root or sudo access
- Knowledge of your network configuration (IP range, gateway, DNS servers)

## Step-by-Step Instructions

### 5. Verify Configuration

Check that your static IP is applied correctly:

```bash
ip addr show ens18
```

Test network connectivity:

```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```

## Configuration Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| Interface | Network interface name | `ens18`, `eth0`, `enp0s3` |
| IP Address | Static IP with subnet mask | `192.168.1.111/24` |
| Gateway | Router IP address | `192.168.1.1` |
| DNS Servers | Domain name servers | `8.8.8.8`, `1.1.1.1` |

## Common Network Interface Names

- **`ens18`** - VMware/VirtualBox virtual machines
- **`eth0`** - Traditional Ethernet naming
- **`enp0s3`** - Physical network interfaces
- **`wlan0`** - Wireless interfaces

## Troubleshooting

### If configuration doesn't apply:

1. Check YAML syntax:
   ```bash
   sudo netplan --debug apply
   ```

2. Restart networking service:
   ```bash
   sudo systemctl restart systemd-networkd
   ```

3. Revert to DHCP if needed:
   ```bash
   sudo netplan --debug generate
   sudo netplan apply
   ```

## Important Notes

- **Backup**: Always backup your original configuration before making changes
- **YAML Syntax**: Be careful with indentation (use spaces, not tabs)
- **Network Range**: Ensure your static IP is within your network's IP range
- **IP Conflicts**: Make sure the IP address isn't already in use by another device

## Example for Different Network

For a different network setup (e.g., 10.0.0.x network):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 10.0.0.100/24
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
``` 1. Identify Your Network Interface

First, identify your network interface name:

```bash
ip addr show
```

Common interface names:
- `ens18` - VMware/VirtualBox virtual machines
- `eth0` - Traditional Ethernet naming
- `enp0s3` - Physical network interfaces

### 2. Edit Netplan Configuration

Open the Netplan configuration file:

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

###
