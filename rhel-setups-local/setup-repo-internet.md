# Offline Repository Setup and Internet Control

This guide shows how to set up offline repositories for AlmaLinux/Rocky Linux and control internet access on your system.

## Step 1: Create Repository Configuration

Create a new repository configuration file:

```bash
sudo vi /etc/yum.repos.d/offline-repo.repo
```

## Step 2: Add Repository Configuration

Add the following content to the file:

```ini
# AlmaLinux BaseOS
[almalinux-baseos]
name=AlmaLinux 8 BaseOS
baseurl=http://repo.almalinux.org/almalinux/8/BaseOS/x86_64/os/
enabled=1
gpgcheck=0

# AlmaLinux AppStream
[almalinux-appstream]
name=AlmaLinux 8 AppStream
baseurl=http://repo.almalinux.org/almalinux/8/AppStream/x86_64/os/
enabled=1
gpgcheck=0

# Rocky Linux BaseOS
[rocky-baseos]
name=Rocky Linux 8 BaseOS
baseurl=http://dl.rockylinux.org/pub/rocky/8/BaseOS/x86_64/os/
enabled=1
gpgcheck=0

# Rocky Linux AppStream
[rocky-appstream]
name=Rocky Linux 8 AppStream
baseurl=http://dl.rockylinux.org/pub/rocky/8/AppStream/x86_64/os/
enabled=1
gpgcheck=0
```

## Step 3: Refresh Repository Cache

Clean the cache and verify repositories:

```bash
sudo dnf clean all
sudo dnf repolist
```

## Step 4: Download RPM Packages

To download packages with all dependencies:

```bash
sudo dnf download --resolve --alldeps --downloaddir=~/offline-rpms firewalld
```

To install packages directly:

```bash
sudo dnf install --nobest --allowerasing -v -y firewalld
```

## Internet Control Setup

### Install iptables-services

First, install the required service:

```bash
sudo dnf install --nobest --allowerasing -v -y iptables-services
```

### Create Internet Control Script

Create the control script:

```bash
sudo vi internet_control.sh
```

Add the following content:

```bash
#!/bin/bash
# File: internet_control.sh
# Usage: sudo ./internet_control.sh block|unblock

SUBNET="192.168.1.0/24"

enable_service() {
    echo "[*] Enabling iptables service..."
    sudo systemctl enable iptables
    sudo systemctl start iptables
}

block_internet() {
    enable_service
    echo "[*] Flushing existing rules..."
    sudo iptables -F
    sudo iptables -X
    sudo iptables -t nat -F
    sudo iptables -t nat -X
    sudo iptables -t mangle -F
    sudo iptables -t mangle -X
    
    echo "[*] Setting default policies to DROP..."
    sudo iptables -P INPUT DROP
    sudo iptables -P FORWARD DROP
    sudo iptables -P OUTPUT DROP
    
    echo "[*] Allowing traffic on local subnet..."
    sudo iptables -A INPUT -s $SUBNET -j ACCEPT
    sudo iptables -A OUTPUT -d $SUBNET -j ACCEPT
    
    echo "[*] Saving rules..."
    sudo service iptables save
    echo "[*] Internet blocked, local subnet allowed."
}

unblock_internet() {
    enable_service
    echo "[*] Flushing existing rules..."
    sudo iptables -F
    sudo iptables -X
    sudo iptables -t nat -F
    sudo iptables -t nat -X
    sudo iptables -t mangle -F
    sudo iptables -t mangle -X
    
    echo "[*] Setting default policies to ACCEPT..."
    sudo iptables -P INPUT ACCEPT
    sudo iptables -P FORWARD ACCEPT
    sudo iptables -P OUTPUT ACCEPT
    
    echo "[*] Saving rules..."
    sudo service iptables save
    echo "[*] Full internet access restored."
}

case "$1" in
    block)
        block_internet
        ;;
    unblock)
        unblock_internet
        ;;
    *)
        echo "Usage: $0 {block|unblock}"
        exit 1
        ;;
esac
```

### Make Script Executable and Use

```bash
chmod +x internet_control.sh
```

### Usage

Block internet access (allow local subnet only):
```bash
sudo ./internet_control.sh block
```

Restore full internet access:
```bash
sudo ./internet_control.sh unblock
```

## Notes

- The script blocks all internet access but allows local subnet communication (192.168.1.0/24)
- Modify the `SUBNET` variable if your network uses a different IP range
- Use with caution as blocking internet will affect system updates and package downloads
