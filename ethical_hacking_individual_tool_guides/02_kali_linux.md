# Kali Linux — Testing Workstation

## 1. Purpose
Kali Linux is a Debian-based distribution with many security-testing, forensics, and network-analysis tools.

**Memory hook:** Kali = **the toolbox**, not the target.

## 2. Inputs
Kali itself receives normal Linux commands, files, target addresses, and tool options. A command's target should be your authorized lab machine or a system you are permitted to assess.

## 3. Essential commands
```bash
whoami
pwd
ip -br addr
ip route
ping -c 4 TARGET_IP
sudo apt update
```
- `whoami`: current user.
- `pwd`: current directory.
- `ip -br addr`: short view of network interfaces and assigned IPs.
- `ip route`: routing table and default gateway.
- `ping`: tests basic IP reachability; firewalls may block ICMP.
- `apt update`: refreshes package indexes; it does not upgrade every installed package.

## 4. Example output and explanation
`ip -br addr` might show:
```text
lo       UNKNOWN  127.0.0.1/8
eth0     UP       10.0.2.15/24
eth1     UP       192.168.56.10/24
```
Interpretation: Kali has two network interfaces. The exact addresses are examples; identify which interface belongs to your Host-only lab network before scanning.

`ping` may show `bytes from TARGET_IP` and round-trip times. That suggests ICMP replies are reaching Kali, but does not prove every service is reachable.

## 5. Common issues
- `Temporary failure resolving`: check DNS and network settings.
- `Permission denied`: some operations need `sudo`.
- `Unable to locate package`: run `sudo apt update`, check repository/network configuration, and verify the package name.
- Wrong target subnet: compare `ip -br addr` and `ip route` before discovery.

## 6. Viva
**Q: Why use Kali?** It bundles security tools in a Linux environment.
**Q: Is Kali itself an exploit?** No; it is an operating system that hosts tools.
**Q: What does `sudo` do?** Runs a command with elevated privileges when authorized.
**Q: Does failed ping prove the host is down?** No; ICMP may be filtered or the route may be wrong.

## 7. Memorize
**Address → Route → Reachability → Tool.** Find your interface, understand the route, check the lab target, then run the relevant tool.
