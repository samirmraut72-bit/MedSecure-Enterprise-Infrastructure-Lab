# 01 — Network Infrastructure

This section presents the network foundation of the MedSecure environment: headquarters and branch connectivity, VLAN segmentation, dynamic routing, gateway redundancy, DHCP, trunks and Layer-2 security.

## Key technologies

- VLANs and 802.1Q trunking
- Inter-VLAN routing
- OSPF
- HSRP
- EtherChannel and STP
- Centralised DHCP
- ACL concepts and port security
- Branch-to-HQ application reachability

The detailed configuration remains in the [Network Administrator repository](https://github.com/samirmraut72-bit/Network-Administrator.).

## Evidence

### Enterprise topology
![Enterprise topology](https://raw.githubusercontent.com/samirmraut72-bit/Network-Administrator./main/screenshots/01-full-topology.png)

### OSPF routing
![OSPF routing](https://raw.githubusercontent.com/samirmraut72-bit/Network-Administrator./main/screenshots/03-ospf-routing.png)

### HSRP gateway redundancy
![HSRP status](https://raw.githubusercontent.com/samirmraut72-bit/Network-Administrator./main/screenshots/04-hsrp-status.png)

### Branch-to-HQ connectivity
![Branch to HQ connectivity](https://raw.githubusercontent.com/samirmraut72-bit/Network-Administrator./main/screenshots/06-branch-hq-ping.png)

### Port security
![Port security](https://raw.githubusercontent.com/samirmraut72-bit/Network-Administrator./main/screenshots/07-port-security.png)
