# Netdiscover — Local Network Host Discovery

## 1. Purpose
Netdiscover uses ARP-based discovery to identify hosts on a local IPv4 Ethernet network. ARP discovery generally works only on the local Layer-2 segment, not across routers.

**Memory hook:** Netdiscover = **who is on this local street?**

## 2. Inputs
- Local interface or local subnet.
- Permission to enumerate that network.
- A Linux environment with the tool installed.

## 3. Commands
```bash
ip -br addr
sudo netdiscover
```
For a known lab subnet, common versions support:
```bash
sudo netdiscover -r LAB_SUBNET/CIDR
```
Check `netdiscover -h` for your version's exact flags.

## 4. Typical output fields
- IP: discovered IPv4 address.
- MAC: hardware address observed on the local segment.
- Count/response data: observations from ARP replies.
- Vendor: manufacturer guess derived from the MAC prefix; may be unknown or inaccurate.

## 5. Explain the output
“The tool observed ARP replies from devices on the local network and listed their IP/MAC pairs. It does not identify every device on routed networks, and vendor identification is only a clue.”

## 6. Troubleshooting
- No hosts: check interface, subnet, VM network mode, and whether hosts are powered on.
- Wrong interface: inspect `ip -br addr`.
- Scanning from NAT: the lab target may not be on the same Layer-2 segment; use the configured Host-only interface.
- No vendor: normal for randomized, virtual, or unrecognized MAC addresses.

## 7. Viva
**Q: What is ARP?** Address Resolution Protocol maps IPv4 addresses to link-layer addresses on a local network.
**Q: Why does it not cross routers?** Routers do not forward ordinary ARP broadcasts.
**Q: Netdiscover vs Nmap?** Netdiscover focuses on local ARP discovery; Nmap supports broader host/port/service scanning.
