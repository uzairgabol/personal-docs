# Ubuntu Static IP Configuration - Step by Step Guide

This guide provides detailed step-by-step instructions to configure a static IP address in Ubuntu using Netplan.

## Step 1: Open Terminal

Press `Ctrl + Alt + T` to open the terminal, or search for "Terminal" in the applications menu.

## Step 2: Check Current Network Configuration

View your current network interfaces and IP addresses:

```bash
ip addr show
```

**Expected Output Example:**
```
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic ens18
```

**Note down:**
- Interface name (e.g., `ens18`, `eth0`, `enp0s3`)
- Current IP address and network range

## Step 3: Check Current Gateway

Find your current gateway (router) IP address:

```bash
ip route show default
```

**Expected Output Example:**
```
default via 192.168.1.1 dev ens18 proto dhcp metric 100
```

**Note down:** Gateway IP (e.g., `192.168.1.1`)

## Step 4: Backup Current Configuration

Create a backup of your current network configuration:

```bash
sudo cp /etc/netplan/01-network-manager-all.yaml /etc/netplan/01-network-manager-all.yaml.backup
```

## Step 5: Open Netplan Configuration File

Edit the Netplan configuration file:

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

**What you'll see:** The nano text editor will open with the current configuration file.

## Step 6: Clear and Replace Configuration

1. **Clear all existing content** by pressing `Ctrl + A` then `Delete`
2. **Copy and paste** the following configuration:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.1.111/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

## Step 7: Customize Your Configuration

**Replace the following values with your specific network settings:**

| Field | What to Replace | Example |
|-------|----------------|---------|
| `ens18` | Your interface name from Step 2 | `eth0`, `enp0s3` |
| `192.168.1.111` | Your desired static IP | `192.168.1.150` |
| `192.168.1.1` | Your gateway IP from Step 3 | `10.0.0.1` |

**Important:** 
- Keep the `/24` after your IP address
- Use spaces for indentation, NOT tabs
- Make sure your static IP is in the same network range as your gateway

## Step 8: Save the Configuration

1. Press `Ctrl + X` to exit
2. Press `Y` to confirm saving
3. Press `Enter` to confirm the filename

## Step 9: Test Configuration Syntax

Check if your YAML syntax is correct:

```bash
sudo netplan --debug generate
```

**What to expect:**
- If successful: No error messages
- If there's an error: Fix the indentation or syntax in the file

## Step 10: Apply the Configuration

Apply your new network settings:

```bash
sudo netplan apply
```

**What happens:** Your network interface will restart with the new static IP.

## Step 11: Verify the Static IP

Check if your static IP is active:

```bash
ip addr show ens18
```

**Replace `ens18`** with your interface name.

**Expected Output:**
```
inet 192.168.1.111/24 brd 192.168.1.255 scope global ens18
```

## Step 12: Test Internet Connectivity

Test if you can reach the internet:

```bash
ping -c 4 8.8.8.8
```

**Expected Output:**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=15.2 ms
```

## Step 13: Test DNS Resolution

Test if domain names resolve correctly:

```bash
ping -c 4 google.com
```

**Expected Output:**
```
PING google.com (142.250.190.78) 56(84) bytes of data.
64 bytes from lga25s62-in-f14.1e100.net (142.250.190.78): icmp_seq=1 ttl=117 time=15.8 ms
```

## Troubleshooting Steps

### If Step 10 (netplan apply) Fails:

**Step A:** Check your YAML syntax
```bash
sudo netplan --debug apply
```

**Step B:** Restart networking service
```bash
sudo systemctl restart systemd-networkd
```

### If You Lose Internet Connection:

**Step A:** Restore backup configuration
```bash
sudo cp /etc/netplan/01-network-manager-all.yaml.backup /etc/netplan/01-network-manager-all.yaml
sudo netplan apply
```

**Step B:** Check if your IP is already in use
```bash
ping 192.168.1.111
```
If you get responses, choose a different IP address.

### If DNS Doesn't Work:

**Step A:** Try different DNS servers in your configuration:
```yaml
nameservers:
  addresses:
    - 1.1.1.1
    - 9.9.9.9
```

## Complete Example Configurations

### For 192.168.1.x Network:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.1.111/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

### For 10.0.0.x Network:
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
```

## Summary

✅ **You have successfully configured a static IP address when:**
- Step 11 shows your desired IP address
- Step 12 successfully pings 8.8.8.8
- Step 13 successfully pings google.com

Your Ubuntu system now has a permanent IP address that won't change when you restart your computer or router.
