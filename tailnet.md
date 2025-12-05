# Enable IP forwarding (required for subnet routing)
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Restart Tailscale with subnet routing enabled
sudo tailscale up --advertise-routes=10.10.40.0/24 --accept-routes
```

### Step 3: Approve Subnet Routes in Tailscale Admin

1. Go to https://login.tailscale.com/admin/machines
2. Find your Proxmox machine in the list
3. Click the "..." menu → Edit route settings
4. **Approve the subnet route** for 10.10.40.0/24
5. (Optional) Enable "Exit node" if you want to route all traffic through Proxmox

### Step 4: Access Proxmox Web UI

Once connected, you can access Proxmox from anywhere using:
```
https://<proxmox-tailscale-ip>:8006
```

Or if you want to use the local IP through the Tailscale tunnel:
```
https://10.10.40.100:8006
