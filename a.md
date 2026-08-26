Ethical Hacking Lab Exam — Complete Prep Guide
ICS1561 / UCS3568 — Covers Experiments 1, 2, 3 (all 12 sub-tools)
Rule that applies to every tool below: only run these against scanme.nmap.org, your own VM/localhost, your own domain, or infra your college has explicitly authorized (e.g. the SSN lab range). Every viva panel opens with "did you have authorization?" — know your answer.

EXPERIMENT 1: Sysmon, theHarvester, NetDiscover, Google Dorking
1a. Sysmon (Windows System Monitor)
What it is: A Windows sysinternals driver+service that logs detailed system activity (process creation, network connects, file creation, registry changes) into the Windows Event Log — way more detail than default Windows auditing.

Setup:

Download Sysmon from Microsoft Sysinternals (Sysmon64.exe).
Get a config XML — the standard one used in labs is SwiftOnSecurity's sysmonconfig-export.xml (defines which events to log).
Install as admin (elevated PowerShell/CMD):
Sysmon64.exe -accepteula -i sysmonconfig-export.xml
Verify service is running:
sc query Sysmon64
Viewing logs:

Open Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon → Operational.
Or via PowerShell:
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
Key Event IDs to know (this is the #1 viva ask):

Event ID	Meaning
1	Process creation
3	Network connection
5	Process terminated
7	Image/DLL loaded
11	File created
12/13/14	Registry create/modify/delete
22	DNS query
Lab demo flow: Open Notepad or ping a host → filter Event Viewer to Sysmon log → find the matching Event ID 1 (process creation, shows parent process, command line, hashes) and Event ID 3 (if network activity) → screenshot both.

Viva questions:

Why Sysmon over default Windows logging? → Default logs don't capture command-line arguments, parent-child process trees, or hashes; Sysmon does — critical for detecting malware/living-off-the-land attacks.
What's in the config XML? → Include/exclude rules that filter noise (e.g., ignore known-good processes) so you don't drown in log volume.
Where do logs get consumed in a real SOC? → Forwarded to a SIEM (Splunk/ELK) via Windows Event Forwarding or an agent.
Difference between Event ID 1 and Event ID 5? → 1 is creation, 5 is termination of a process.
1b. theHarvester (passive OSINT — emails/subdomains)
What it is: Passive recon tool that queries public sources (search engines, certificate transparency logs, DNS) to harvest emails, subdomains, hostnames, and employee names for a target domain — without touching the target directly.

Install (Kali has it preinstalled; else):

sudo apt install theharvester -y
# or
pip install theHarvester
Core command:

theHarvester -d example.com -b google -l 200
-d → target domain
-b → source (google, bing, duckduckgo, crtsh, hackertarget, etc. — or -b all)
-l → limit number of results
-f report.html → save output as HTML report
Good demo command:

theHarvester -d ssn.edu.in -b crtsh,hackertarget -l 100 -f ssn_report
Output interpretation: You'll get lists of emails found, subdomains/hosts (with IPs if resolvable), and sometimes LinkedIn employee names if -b linkedin works. This maps your target's attack surface / social engineering surface.

Viva questions:

Passive vs active recon? → Passive = no packets sent to target, only public sources queried (theHarvester, WHOIS). Active = you touch the target (Nmap, Nikto).
Why is this legal even without permission (usually)? → Because it only queries publicly indexed data, not the target itself — but scope/authorization still matters for coursework/CTF rules.
What's crt.sh source? → Certificate Transparency log search — finds subdomains from SSL cert issuance records, very reliable for subdomain enumeration.
Real-world use by attackers? → Building phishing target lists (emails) and initial attack surface mapping (subdomains).
1c. NetDiscover (active LAN host discovery)
What it is: Sends ARP requests across a local subnet to discover live hosts and their MAC addresses — layer 2 discovery, works even when hosts block ICMP ping.

Install: preinstalled on Kali; else sudo apt install netdiscover

Core commands:

sudo netdiscover -i eth0            # passive listen mode, auto-detect range
sudo netdiscover -r 192.168.1.0/24  # active scan of a specific range
sudo netdiscover -P                 # print results and exit (non-interactive, good for reports)
Output columns: IP address | MAC address | Count | Len | MAC Vendor / Hostname

Viva questions:

Why ARP instead of ICMP ping? → ARP works only within the local broadcast domain but can't be firewalled off like ICMP — every device MUST respond to ARP to communicate at all, so it's more reliable for LAN discovery.
Can NetDiscover cross subnets/through a router? → No — ARP doesn't route; it's confined to the local L2 segment.
How would you identify device type from output? → MAC vendor OUI (first 3 bytes) maps to manufacturer (e.g., a MAC starting with a known Cisco/Apple/Raspberry Pi prefix).
Passive vs active mode difference? → Passive just sniffs existing ARP traffic on the wire; active mode sends ARP requests to force replies (more complete but generates traffic/noise).
1d. Google Dorking (advanced search operators)
What it is: Using Google's advanced search syntax to surface publicly indexed but sensitive/unintended content (login portals, exposed files, misconfigured directories) about a target.

Key operators (memorize these — always asked):

Operator	Purpose	Example
site:	Restrict to a domain	site:example.com
filetype:	Search specific file type	filetype:pdf site:example.com
intitle:	Text in page title	intitle:"index of"
inurl:	Text in URL	inurl:admin login
intext:	Text in page body	intext:"password"
-	Exclude term	site:example.com -www
"..."	Exact phrase	"confidential" filetype:xlsx
cache:	Google's cached version	cache:example.com
High-value combos to demo:

site:example.com filetype:pdf
site:example.com inurl:admin
intitle:"index of" site:example.com
site:example.com ext:sql | ext:env | ext:log
Viva questions:

Is Google Dorking hacking? → No exploitation happens — it's passive OSINT using a search engine's own indexing features; it's legal to search, but acting on found sensitive data without authorization is not.
What is the Google Hacking Database (GHDB)? → Exploit-DB's curated collection of known effective dork strings for finding vulnerable systems/exposed data.
How does intitle:"index of" help attackers? → It finds open/unprotected directory listings (misconfigured web servers exposing raw file trees).
Mitigation for organizations? → robots.txt, proper access controls, removing sensitive docs from public web roots, periodically dorking your own domain to audit exposure.
EXPERIMENT 2: FOCA, WHOIS, nsLookup, Sublist3r
2a. FOCA (metadata extraction from public documents)
What it is: A Windows GUI tool that crawls a domain (via search engines) for public documents (PDF, DOCX, XLSX, PPTX) and extracts embedded metadata — usernames, software versions, internal folder paths, printer names, emails.

Workflow:

Open FOCA → New Project → enter domain (e.g., example.com).
Choose search engines (Google/Bing) and file extensions (pdf, doc, xls, ppt).
Click Search All → FOCA lists discovered documents.
Right-click documents → Download → Extract Metadata.
FOCA parses metadata into categories: Users, Folders, Software, Printers, Emails, OS.
Viva questions:

What metadata is most dangerous? → Usernames (reveal naming convention for password spraying), software versions (reveal unpatched vulnerable versions), and internal folder paths (reveal internal network/server structure).
How is metadata created in a Word doc? → Automatically embedded by the authoring software (author name = OS username, "last saved by," software version, revision history) unless scrubbed.
Mitigation? → Use "Document Inspector"/metadata-removal tools before publishing files publicly, or strip metadata server-side before upload.
Difference from theHarvester? → theHarvester pulls from search-engine-indexed text/DNS; FOCA specifically downloads and parses document metadata, a different data class entirely.
2b. WHOIS (domain registration lookup)
What it is: Queries WHOIS databases (maintained by registrars/registries) for domain registration details.

Command:

whois example.com
(Install if missing: sudo apt install whois)

Key fields to point out in output:

Registrar name
Creation date / Expiry date / Updated date
Name servers (NS records)
Registrant/Admin/Tech contact (often redacted by GDPR privacy proxies now)
Domain status codes (e.g., clientTransferProhibited)
Viva questions:

Why is registrant info often hidden today? → GDPR/ICANN privacy protection — registrars use proxy/redaction services to hide personal data of domain owners.
What can an attacker do with WHOIS data? → Social engineering (calling registrar pretending to be the owner), domain expiry monitoring for domain hijacking, or identifying related infrastructure via registrant email reuse.
Difference between WHOIS and nsLookup? → WHOIS gives ownership/registration data; nsLookup gives live DNS resolution data (IPs, records) — different databases entirely.
2c. nsLookup (DNS reconnaissance)
What it is: Queries DNS servers for various record types about a domain.

Commands:

nslookup example.com                  # default A record lookup
nslookup -type=MX example.com         # mail servers
nslookup -type=NS example.com         # name servers
nslookup -type=TXT example.com        # TXT records (SPF, verification, etc.)
nslookup -type=ANY example.com        # all records (often restricted now)
nslookup example.com 8.8.8.8          # query using Google's DNS explicitly
Interactive mode:

nslookup
> server 8.8.8.8
> set type=MX
> example.com
Record types to know:

Record	Meaning
A	Hostname → IPv4
AAAA	Hostname → IPv6
MX	Mail exchange servers
NS	Authoritative name servers
TXT	Arbitrary text (SPF/DKIM/domain verification)
CNAME	Alias to another hostname
PTR	Reverse lookup (IP → hostname)
Viva questions:

Why would MX records matter to an attacker? → Identifies the mail provider (e.g., Google Workspace/Office365) → informs phishing infrastructure targeting.
What's a zone transfer (AXFR) and why is it dangerous? → A full DNS zone transfer dumps every record in the domain if misconfigured to allow it to anyone — huge info leak; test with dig axfr @nameserver domain.com.
dig vs nslookup? → dig gives more raw/detailed output and is generally preferred by professionals; nslookup is simpler/legacy but still common in labs.
2d. Sublist3r (passive subdomain enumeration)
What it is: Python tool that aggregates subdomains from multiple passive sources (Google, Bing, Yahoo, Baidu, VirusTotal, DNSdumpster, Netcraft) in one shot.

Install:

git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r
pip install -r requirements.txt
Core command:

python3 sublist3r.py -d example.com
python3 sublist3r.py -d example.com -o subdomains.txt   # save output
python3 sublist3r.py -d example.com -b                  # include brute-force module
python3 sublist3r.py -d example.com -p 80,443           # port scan discovered subdomains
Viva questions:

Why enumerate subdomains? → Subdomains often run older/forgotten apps (dev, staging, test environments) with weaker security — classic attacker entry point.
Passive vs brute-force subdomain enum? → Passive = query existing public sources (fast, stealthy, may miss unlisted ones); brute-force = try a wordlist of common subdomain names against DNS (slower, noisier, finds unlisted ones).
Tools that do similar things? → Amass, Assetfinder, crt.sh, Subfinder — Sublist3r is the "classic" one taught in most labs.
EXPERIMENT 3: Maltego, Shodan, Nikto, Nmap
3a. Maltego (OSINT visualization / link analysis)
What it is: A GUI OSINT tool that represents entities (domains, IPs, emails, people, organizations) as nodes and runs "Transforms" (automated queries to external data sources) to expand relationships into a graph.

Workflow:

Open Maltego → New Graph.
Drag a Domain entity onto canvas → type your target domain.
Right-click → Run Transform → e.g., "To DNS Name," "To Email addresses," "To Website."
Keep expanding nodes (subdomains → IPs → hosting providers → related domains) to build the relationship graph.
Export/screenshot the final graph for your report.
Viva questions:

What is a "Transform"? → A modular query (often via API — e.g., to Shodan, VirusTotal, WHOIS) that takes an input entity and returns related entities.
Community Edition (CE) vs paid? → CE has a limited transform set and result caps (usually 12 results/transform); paid unlocks full commercial transform hubs and higher limits.
Why is graph visualization valuable over raw text output (like theHarvester)? → It reveals relationships/pivots — e.g., shared infrastructure between seemingly unrelated domains — that flat lists don't show.
Real use case? → Threat intel analysts mapping an adversary's infrastructure, or red teamers mapping an org's full digital footprint before an engagement.
3b. Shodan (internet-wide device/service search engine)
What it is: A search engine that continuously scans the entire IPv4 space and indexes banners (service responses) — letting you search for exposed devices/services by software, version, port, geography, or organization, without touching the target yourself.

Usage (web UI: shodan.io):

Basic query: apache city:"Chennai"
By org: org:"SSN College of Engineering"
By port: port:22
By product/version: product:"nginx" version:"1.18"
By vulnerability: vuln:CVE-2021-44228 (Log4Shell — needs a paid/academic plan for full vuln search)
CLI (if installed):

pip install shodan
shodan init YOUR_API_KEY
shodan host 8.8.8.8          # info on a specific IP
shodan search "apache country:IN"
Viva questions:

How is Shodan different from Google? → Google indexes web page content; Shodan indexes service banners from any internet-facing port (SSH, RDP, SCADA/ICS, webcams, databases) — it's a search engine for the internet's infrastructure, not its content.
Why is Shodan controversial? → It surfaces unsecured/misconfigured devices (open webcams, exposed industrial control systems) to anyone searching — a double-edged tool for both defenders and attackers.
Legal to search Shodan? → Yes, searching is passive (you're querying Shodan's database, not scanning the target yourself); acting on exposed access without authorization is illegal.
Defensive use? → Organizations run Shodan queries on their own IP ranges to find their own accidental exposures (shadow IT, forgotten servers).
3c. Nikto (web server vulnerability scanner)
What it is: An active scanner that sends thousands of requests to a web server checking for outdated software versions, dangerous files/CGIs, misconfigurations, and known vulnerable server signatures.

Install: preinstalled on Kali; else sudo apt install nikto

Core commands:

nikto -h http://scanme.nmap.org
nikto -h 192.168.1.10 -p 80,443          # specific ports
nikto -h example.com -ssl                # force SSL
nikto -h example.com -o report.html -Format html   # save report
nikto -h example.com -Tuning 1,2,3       # tune scan type (see below)
Tuning options (worth knowing 2-3): 1=file upload, 2=misconfig, 3=info disclosure, 6=XSS, 9=SQLi

Output interpretation: Nikto flags things like: outdated server banner (Server: Apache/2.2.8 = old/vulnerable), missing security headers (X-Frame-Options not present), default/dangerous files found (/admin/, /phpinfo.php), directory indexing enabled.

Viva questions:

Is Nikto active or passive? → Active — it sends real HTTP requests directly to the target, so it's noisy and easily logged/detected (unlike Ch1-2 tools).
What kind of vulnerabilities does Nikto NOT find? → Logic flaws, authentication bypass, business-logic issues — it's signature/pattern-based, not intelligent; it checks known bad configs/old versions, not custom app logic.
False positives? → Common — Nikto flags based on version banners which can be spoofed or outdated without actually being vulnerable; results need manual verification.
Nikto vs Nessus/OpenVAS? → Nikto is web-server-specific and lightweight; Nessus/OpenVAS are full vulnerability scanners covering network services, OS, and CVEs broadly.
3d. Nmap (active network/port scanning) — the big one, expect the most viva depth here
What it is: The industry-standard active scanner for host discovery, port scanning, service/version detection, and OS fingerprinting.

Core scan types (know what each ACTUALLY does at the packet level):

nmap -sn 192.168.1.0/24        # Ping scan / host discovery only (no port scan)
nmap -sS target                # SYN scan ("half-open" / stealth scan) — default, needs root
nmap -sT target                # TCP connect scan (full 3-way handshake, no root needed)
nmap -sU target                # UDP scan
nmap -sV target                # Service/version detection
nmap -O target                 # OS detection
nmap -A target                 # Aggressive: OS+version+script+traceroute combined
nmap -p- target                # All 65535 ports
nmap -p 22,80,443 target       # Specific ports
nmap -T4 target                # Timing template (0=slowest/stealthiest, 5=fastest)
nmap -oN out.txt target        # Normal output to file (-oX for XML, -oA for all formats)
nmap --script vuln target      # NSE vulnerability scripts
Good demo combo for the exam:

sudo nmap -sS -sV -O -p- -T4 scanme.nmap.org -oN nmap_report.txt
Port states (always asked):

State	Meaning
open	Service actively accepting connections
closed	Port reachable but no service listening
filtered	Firewall/ACL is blocking — Nmap can't tell open/closed
unfiltered	Reachable but state unknown (used in ACK scan)
How SYN scan actually works (classic viva question):

Nmap sends a SYN packet.
If target replies SYN-ACK → port is open → Nmap sends RST (never completes handshake — hence "stealth"/"half-open").
If target replies RST → port is closed.
If no reply / ICMP unreachable → port is filtered.
Viva questions:

SYN scan vs Connect scan — why does SYN need root? → SYN scan crafts raw packets directly (needs raw socket privilege = root); Connect scan uses the OS's normal connect() system call, which any user can do, but it completes the full handshake so it's more easily logged.
How does OS fingerprinting (-O) work? → Nmap examines subtle differences in how different OS TCP/IP stacks respond (TTL values, window size, TCP options ordering) and matches against its fingerprint database.
What is NSE (Nmap Scripting Engine)? → Lua-based scripting framework (--script) enabling extended functionality — vuln detection, brute force, malware detection, etc. (e.g. --script http-title, --script vuln).
Difference between -sV and -A? → -A is a convenience flag that bundles -sV (version detect) + -O (OS detect) + default NSE scripts + traceroute all together.
Why is UDP scanning slower/less reliable than TCP? → UDP is connectionless — no response from an open UDP port (many services don't reply to empty probes) means Nmap can't distinguish open vs filtered without waiting for ICMP port-unreachable timeouts.
How would a firewall show up in Nmap results? → Ports show as filtered instead of closed — Nmap gets no response or an ICMP admin-prohibited message rather than a RST.
Cross-Cutting Viva Questions (asked regardless of which experiment you get)
Passive vs Active recon — classify every tool:

Passive: theHarvester, WHOIS, nsLookup*, Sublist3r, Google Dorking, Shodan, FOCA (search-engine crawl phase)
Active: NetDiscover, Nmap, Nikto, Sysmon (local monitoring), Maltego (some transforms hit target directly)
(nsLookup is passive toward the target — you're querying a DNS resolver, not the target itself)
The 5 phases of ethical hacking — know these cold:

Reconnaissance (passive + active) — everything in this lab
Scanning & Enumeration — Nmap, Nikto
Gaining Access — exploitation (not covered in this lab)
Maintaining Access — persistence (not covered)
Covering Tracks / Reporting — final report
Why does authorization/scope matter legally? → Under the IT Act 2000 (India) and most countries' computer misuse laws, unauthorized scanning/access is a criminal offense regardless of intent — a signed scope document ("Rules of Engagement") is what makes any of this "ethical."

CIA Triad relevance to recon → Recon primarily threatens Confidentiality first (info disclosure) — it's the reconnaissance stage that later enables Integrity/Availability attacks.

Report structure they'll expect from your lab writeup: Objective → Tool/Version → Command used → Screenshot of output → Interpretation/findings → Mitigation recommendation. Always include the mitigation line per tool — examiners specifically check for it.

Quick Pre-Viva Checklist
 Can state passive vs active for every single tool
 Can explain one command's flags without looking (pick your strongest tool)
 Know at least 1 real Event ID (Sysmon), 1 real DNS record type, 1 Nmap port state, 1 Google dork operator, cold
 Have a mitigation/defense answer ready for at least 3 tools
 Know the 5 phases of ethical hacking in order
 Screenshots ready for whichever experiment you actually run in the lab