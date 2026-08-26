# Ethical Hacking Lab — Syntax Cheat Sheet (All 12 Tools)

## Experiment 1

### Sysmon (Windows / PowerShell)
```
Sysmon64.exe -accepteula -i sysmonconfig-export.xml   # install with config
sc query Sysmon64                                       # verify service running
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

### theHarvester
```
theHarvester -d example.com -b google -l 200
theHarvester -d example.com -b crtsh,hackertarget -l 100 -f report
```
`-d` domain | `-b` source (or `all`) | `-l` limit | `-f` output filename (NOT `-o`)

### NetDiscover
```
sudo netdiscover -i eth0            # passive listen
sudo netdiscover -r 192.168.1.0/24  # active scan of range
sudo netdiscover -P                 # print & exit (non-interactive)
```

### Google Dorking (typed into Google, not terminal)
```
site:example.com filetype:pdf
site:example.com inurl:admin
intitle:"index of" site:example.com
site:example.com -www
"exact phrase" filetype:xlsx
```

---

## Experiment 2

### FOCA (GUI — no CLI syntax)
Project → New Project → enter domain → check search engines + extensions → **Search All** → right-click file(s) → **Download** → right-click → **Extract Metadata**.
Use **Custom search** to manually feed URLs if live search returns 0 results.

### WHOIS
```
whois example.com
```

### nsLookup
```
nslookup example.com                        # default A record
nslookup -type=A example.com                 # IPv4 record
nslookup -type=AAAA example.com              # IPv6 record
nslookup -type=MX example.com                # mail servers
nslookup -type=NS example.com                # name servers
nslookup -type=TXT example.com               # SPF/DKIM/verification
nslookup -type=ANY example.com               # all records (often restricted)
nslookup example.com 8.8.8.8                 # query via specific DNS server (no flag, just append)
```
Interactive mode:
```
nslookup
> server 8.8.8.8
> set type=MX
> example.com
```

### Sublist3r
```
python3 sublist3r.py -d example.com
python3 sublist3r.py -d example.com -o subdomains.txt
python3 sublist3r.py -d example.com -b            # include brute-force
python3 sublist3r.py -d example.com -p 80,443     # port scan discovered subs
```
(If Kali's apt package added a system binary, plain `sublist3r -d example.com` also works — know both forms.)

---

## Experiment 3

### Maltego (GUI — no CLI syntax)
New Graph → drag **Domain** entity → type target → right-click → **Run Transform** (e.g. "To DNS Name," "To Email addresses").

### Shodan
```
# Web UI query syntax (shodan.io):
apache city:"Chennai"
org:"Target Org"
port:22
product:"nginx" version:"1.18"
vuln:CVE-2021-44228

# CLI:
pip install shodan
shodan init YOUR_API_KEY
shodan host 8.8.8.8
shodan search "apache country:IN"
```

### Nikto
```
nikto -h http://target.com
nikto -h target.com -p 80,443
nikto -h target.com -ssl
nikto -h target.com -o report.html -Format html   # -o is correct HERE (Nikto only)
nikto -h target.com -Tuning 1,2,3
```

### Nmap
```
nmap -sn 192.168.1.0/24        # host discovery only, no ports
nmap -sS target                 # SYN stealth scan (needs root)
nmap -sT target                 # TCP connect scan (no root needed)
nmap -sU target                 # UDP scan
nmap -sV target                 # service/version detection
nmap -O target                  # OS detection
nmap -A target                  # -sV + -O + default scripts + traceroute, bundled
nmap -p- target                 # all 65535 ports
nmap -p 22,80,443 target        # specific ports
nmap -T4 target                 # faster timing template
nmap -oN out.txt target         # save NORMAL output — must be -oN, not -o
nmap -oX out.xml target         # save XML output
nmap -oA out target             # save all formats at once
nmap --script vuln target       # NSE vulnerability scripts
```
Full demo combo:
```
sudo nmap -sS -sV -O -p- -T4 scanme.nmap.org -oN nmap_report.txt
```

---

## ⚠️ Output-flag trap (mixes people up most)
| Tool | Save-output flag |
|---|---|
| Nmap | `-oN` (normal) / `-oX` (XML) / `-oA` (all) — never plain `-o` |
| theHarvester | `-f` |
| Nikto | `-o` (with `-Format html/xml/csv`) |
| Sublist3r | `-o` |

There is no universal "save output" flag across tools — check each one individually.
