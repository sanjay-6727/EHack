# Ethical Hacking Tool Guides — Start Here

This pack contains one memorable, practical Markdown guide per tool or lab component.

## The 20 guides
1. VirtualBox
2. Kali Linux
3. Metasploitable 2
4. Armitage
5. Metasploit Framework
6. Nmap
7. WHOIS
8. nslookup
9. theHarvester
10. Netdiscover
11. Sublist3r
12. Wireshark
13. Nessus Essentials
14. Sysmon
15. FOCA
16. Maltego
17. Robtex
18. Shodan
19. Google Dorking
20. PostgreSQL for Metasploit/Armitage

## How to study each guide
Remember **I-C-O-E-V**:
- **I — Input:** What do I give the tool?
- **C — Command/clicks:** How do I run it?
- **O — Output:** What appears?
- **E — Explain:** What does the result mean?
- **V — Viva:** What might the examiner ask?

## Safe lab rule
Use active scans and testing only against your own lab machines or systems for which you have explicit permission. Keep Metasploitable 2 on a VirtualBox **Host-only** network; do not bridge it to your normal network or expose it to the internet. Replace example IPs/domains with your own lab values. Outputs vary by version, configuration, permissions, and network.

## Recommended lab flow
1. Start VirtualBox and boot Kali.
2. Boot Metasploitable 2 on the same Host-only network.
3. Find the target IP using `ip addr` on the target or `ip -br addr` on Kali.
4. From Kali, test reachability with `ping -c 4 TARGET_IP`.
5. Discover lab hosts with `sudo nmap -sn LAB_SUBNET/CIDR`.
6. Scan the lab target with `nmap -sV TARGET_IP`.
7. Capture traffic in Wireshark if required.
8. If using Armitage, check PostgreSQL and the Metasploit database first.
9. Record the **input, exact command, key output, interpretation, and conclusion** in your lab notebook.

## Output explanation template
> “I supplied **[input]** to **[tool]**. The tool returned **[important result]**. This indicates **[meaning]**. It does not automatically prove **[limitation]**. I would verify it by **[safe follow-up]**.”
