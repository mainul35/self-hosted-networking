# OpenVPN Client Configuration Guide for Ubuntu

This guide covers how to replace an existing OpenVPN client profile or configure a new one using systemd on Ubuntu.

## Prerequisites

- Ubuntu with OpenVPN installed (`sudo apt install openvpn`)
- Your `.ovpn` profile file
- Credentials file (if using username/password authentication)

## Directory Structure

OpenVPN client configurations are stored in:

```
/etc/openvpn/client/
```

## Step 1: Prepare Your Files

Copy your OpenVPN profile and credentials to the client directory:

```bash
sudo cp /path/to/your-profile.ovpn /etc/openvpn/client/
sudo cp /path/to/credentials.txt /etc/openvpn/client/
```

Set proper permissions:

```bash
sudo chmod 600 /etc/openvpn/client/credentials.txt
sudo chmod 600 /etc/openvpn/client/your-profile.ovpn
```

## Step 2: Rename the Profile

The systemd service expects a `.conf` extension. Rename your `.ovpn` file:

```bash
sudo mv /etc/openvpn/client/your-profile.ovpn /etc/openvpn/client/myvpn.conf
```

> **Note:** The filename (without `.conf`) becomes the service name. For example, `myvpn.conf` corresponds to `openvpn-client@myvpn`.

## Step 3: Configure Credentials

Edit the configuration file:

```bash
sudo nano /etc/openvpn/client/myvpn.conf
```

Find the `auth-user-pass` line and update it to point to your credentials file:

```
auth-user-pass /etc/openvpn/client/credentials.txt
```

The credentials file should contain two lines:

```
your-username
your-password
```

## Step 4: Manage the Service

### Stop the old VPN service (if replacing)

```bash
sudo systemctl stop openvpn-client@old-vpn-name
sudo systemctl disable openvpn-client@old-vpn-name
```

### Start the new VPN service

```bash
sudo systemctl start openvpn-client@myvpn
```

### Enable auto-start on boot

```bash
sudo systemctl enable openvpn-client@myvpn
```

### Check status

```bash
sudo systemctl status openvpn-client@myvpn
```

### View logs

```bash
sudo journalctl -u openvpn-client@myvpn -f
```

## Step 5: Clean Up Old Profiles (Optional)

Remove old configuration files if no longer needed:

```bash
sudo rm /etc/openvpn/client/old-profile.conf
```

## Troubleshooting

### "Too many open files" Error

If you encounter this error when restarting the service:

```
Failed to allocate directory watch: Too many open files
```

Increase the inotify watch limits:

```bash
sudo sysctl fs.inotify.max_user_watches=524288
sudo sysctl fs.inotify.max_user_instances=512
```

To make this permanent, add to `/etc/sysctl.conf`:

```bash
echo "fs.inotify.max_user_watches=524288" | sudo tee -a /etc/sysctl.conf
echo "fs.inotify.max_user_instances=512" | sudo tee -a /etc/sysctl.conf
```

### Connection Issues

1. Verify the configuration syntax:
   ```bash
   sudo openvpn --config /etc/openvpn/client/myvpn.conf
   ```

2. Check if credentials file path is correct and readable

3. Ensure firewall rules allow VPN traffic

### Service Won't Start

Check detailed logs:

```bash
sudo journalctl -u openvpn-client@myvpn --no-pager -n 50
```

## Quick Reference

| Action | Command |
|--------|---------|
| Start VPN | `sudo systemctl start openvpn-client@myvpn` |
| Stop VPN | `sudo systemctl stop openvpn-client@myvpn` |
| Restart VPN | `sudo systemctl restart openvpn-client@myvpn` |
| Check Status | `sudo systemctl status openvpn-client@myvpn` |
| Enable on Boot | `sudo systemctl enable openvpn-client@myvpn` |
| Disable on Boot | `sudo systemctl disable openvpn-client@myvpn` |
| View Logs | `sudo journalctl -u openvpn-client@myvpn -f` |
