Document how you combined PVE1 and PVE2 into one Proxmox cluster.

Include:
- What is a Proxmox cluster?
- Cluster architecture.
- Creating the cluster on PVE1.
- Adding PVE2 to the cluster.
- Configuring node communication.
- Verifying cluster status.
- Understanding quorum and QDevice.
Commands:
# PVE1
pvecm create mycluser

# PVE2
pvecm add 192.168.122.27

# Verification
pvecm status
pvecm nodes
