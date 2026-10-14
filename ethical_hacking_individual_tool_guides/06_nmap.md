# Nmap — Network Discovery and Port/Service Scanning

## 1. Purpose
Nmap identifies hosts and examines network ports/services. It is an active scanner: it sends network probes.

**Memory hook:** Nmap asks: **Who is there? Which doors answer? What service is behind them?**

## 2. Inputs
- Target IP/hostname or authorized subnet.
- Scan options (discovery, port range, version detection).
- Permissions and network route.
Use `TARGET_IP` below as a placeholder, not a literal address.

## 3. Core commands
```bash
nmap -h
nmap -sn LAB_SUBNET/CIDR
nmap TARGET_IP
nmap -sV TARGET_IP
nmap -p 22,80,443 TARGET_IP
sudo nmap -O TARGET_IP
```
- `-sn`: host discovery without a port scan.
- default scan: scans a default set of common ports.
- `-sV`: probes open ports to estimate service/version.
- `-p`: selects ports.
- `-O`: OS detection; usually needs elevated privileges and may be uncertain.

## 4. Example output
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH ...
80/tcp open  http    Apache httpd ...
```
This is illustrative; your target and versions will differ.

## 5. Explain the output
- **PORT:** transport port and protocol.
- **STATE:** `open` means an application accepted/provided a response consistent with listening; `closed` means reachable but no service listening; `filtered` means Nmap cannot determine state due to filtering/no response.
- **SERVICE:** likely service associated with the port.
- **VERSION:** best-effort fingerprint, not guaranteed exact.
An open port is not automatically a vulnerability.

## 6. Common problems
- `Host seems down`: host may be offline, routed incorrectly, or filtering discovery probes. Verify IP/network first.
- Many `filtered` ports: firewall or network filtering may be involved.
- Version not detected: service may suppress probes or use an unusual configuration.
- Scan is slow: reduce scope to the needed ports and use modest timing in the lab.

## 7. Lab workflow
1. `ip -br addr` to identify Kali's lab interface.
2. `ping -c 4 TARGET_IP` as a basic reachability check.
3. `nmap -sn LAB_SUBNET/CIDR` to discover lab hosts.
4. `nmap -sV TARGET_IP` to inspect services.
5. Record exact command and notable ports; validate findings separately.

## 8. Viva
**Q: TCP vs UDP scan?** They probe different transport protocols; UDP responses are often less direct.
**Q: Open vs filtered?** Open indicates a responding service; filtered means the scanner cannot determine state because probes are blocked or unanswered.
**Q: Does Nmap exploit vulnerabilities?** Nmap is primarily for discovery and scanning, though it includes scripting capabilities.
**Q: Why use `-sV`?** To estimate service and version.
