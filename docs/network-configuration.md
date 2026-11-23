Explain how you configured communication between the two Proxmox nodes.

Include:
- VirtualBox network configuration.
- Network mode used (NAT Network, Host-only, Bridged, etc.).
- Proxmox network interfaces (vmbr0).
- Static IP configuration.
- Hostname configuration.
- /etc/hosts configuration.
- Connectivity tests using ping.
- SSH connectivity tests.
Important commands:
ip -br addr
ip route
hostnamectl
ping 192.168.122.27
ping 192.168.122.37
