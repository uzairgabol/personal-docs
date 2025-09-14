```bash
sudo systemctl stop k3s-agent
sudo systemctl stop rancher-system-agent
sudo rm -rf /etc/rancher/agent /var/lib/rancher/agent /var/lib/rancher/system-agent
sudo rm -rf /etc/rancher/node /var/lib/rancher/k3s
sudo systemctl daemon-reload
```

```bash
sudo hostnamectl set-hostname ubuntutwo-01
```

```bash
sudo journalctl -u rancher-system-agent -f

```

View Logs
```bash
sudo journalctl -u k3s-agent -f
sudo systemctl status k3s-agent
```
