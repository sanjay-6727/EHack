# Metasploitable 2 — Deliberately Vulnerable Target

## 1. Purpose
Metasploitable 2 is an intentionally vulnerable Linux VM designed for safe security-training exercises.

**Memory hook:** Metasploitable = **the practice dummy**.

## 2. Inputs
- The imported VM image.
- A safe Host-only or Internal Network configuration.
- A login for the VM console if needed.
- The target IP address for tools run from Kali.

The commonly documented training login is `msfadmin` / `msfadmin`; use it only inside your own isolated lab and change/avoid exposing it.

## 3. Start-up and IP discovery
1. Import the VM into VirtualBox.
2. Set its network adapter to the same Host-only network as Kali.
3. Boot it and log in at the console.
4. Try:
```bash
ifconfig
ip addr
```
Find the IPv4 address on the lab interface. Do not copy an example address blindly.

From Kali:
```bash
ip -br addr
ping -c 4 TARGET_IP
sudo nmap -sn LAB_SUBNET/CIDR
```

## 4. Typical observations
- `inet 192.168.x.y`: an IPv4 address assigned to an interface (example only).
- Ping replies: the target is responding to ICMP.
- Nmap discovery result: a host was detected using the probes that were sent.

**Explain:** “I booted the intentionally vulnerable target on an isolated network, identified its lab IP, and verified connectivity before running any assessment.”

## 5. Safety checks
- Do not use Bridged networking.
- Do not configure router port forwarding to this VM.
- Do not scan public IPs while practicing.
- If unsure about the network mode, power off the VM and inspect VirtualBox settings first.

## 6. Troubleshooting
- **No address:** check adapter is enabled, DHCP/network configuration, and matching Host-only network.
- **Kali cannot reach target:** confirm both are on the same subnet and firewall/adapter settings.
- **Target appears on home LAN:** stop and correct the network mode.

## 7. Viva
**Q: Why is it used?** To practice assessment against a target designed to be vulnerable.
**Q: Why isolate it?** Its old software and weak configurations are unsafe on ordinary networks.
**Q: Is it a production server?** No.
**Q: What is the first step before scanning?** Confirm authorization, target IP, and network isolation.
