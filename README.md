Standalone terminal tools for server setup. The Xray installers configure Debian 13 (x86-64 or ARM64) for direct VLESS connections or a two-server VLESS proxy chain.

## Installation

Both servers require clean Debian 13 (Trixie) on x86-64 or ARM64, root, systemd, GRUB, an IPv4 default route and the `curl`, `update-grub`, `ip`, `systemctl`, `python3` and `flock` commands. Point each domain's A record to its server and remove its AAAA record. Allow inbound TCP port 80 for the ACME challenge and TCP port 443 for VLESS and HTTPS.

Run the installers as root from the server console or over IPv4 SSH. Both disable IPv6 during setup and remove the SSH server and reboot after successful checks. Keep console access available.

### Main server

The main server requires Netplan with `systemd-networkd` and persistent configuration in `/etc/netplan`.

```bash
curl -4fsSL https://raw.githubusercontent.com/sashokey/terminal-scripts/master/xray-install/setup.sh -o setup.sh && bash setup.sh
```

Enter the main server's domain and generate or enter a UUID. Save the VLESS link displayed by the installer.

### Proxy server (slave)

Set up the proxy server to route `client -> slave -> main`. The slave supports Netplan with `systemd-networkd` or ifupdown with static IPv4. The download command uses `wget`.

```bash
wget -qO setup-proxy.sh https://raw.githubusercontent.com/sashokey/terminal-scripts/master/xray-install/setup-proxy.sh && bash setup-proxy.sh
```

Enter the slave domain, generate or enter a UUID, and paste the main server's VLESS link. Save the displayed slave VLESS link and use it to connect through both servers.
