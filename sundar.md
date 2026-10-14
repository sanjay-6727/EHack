# 🔐 Ethical Hacking Lab Exam – Complete Study Guide
## ICS1561 / UCS3568 | Practical Exam Preparation | 100-Mark Model Exam | 16 Core Tools

> [!IMPORTANT]
> **All tools must be used ONLY on authorized targets/networks you have explicit permission to test.**
> Using these tools on unauthorized systems is illegal.

> **How to use this guide:** Your original tools are retained and reviewed, then expanded with Wireshark, Robtex, Nessus, and Armitage. Use the commands and UI paths as practical starting points; interface labels, installation requirements, licensing limits, and sample output can vary by version. The examples are illustrative—use actual lab IPs and record only the output you observe.
> **Placeholder rule:** Any value inside angle brackets, such as `<authorized-subnet>` or `<supported-backend>`, is a placeholder. Replace it with the real value before running the command; do not paste the angle brackets literally.

## Platform map for your lab setup

Use this map in the exam so you open the tool in the expected environment. A tool may support other platforms, but this guide follows the arrangement you specified.

| Platform | Tools | How you normally use them |
|---|---|---|
| **Windows** | **FOCA, Nessus, Sysmon, Maltego** | FOCA and Maltego are desktop GUI applications; Nessus is operated through a browser UI hosted by the installed Windows service; Sysmon is configured with an elevated terminal and reviewed in Event Viewer/PowerShell. |
| **Kali Linux** | **WHOIS, nslookup, Nmap, Armitage, theHarvester, Netdiscover, Sublist3r, Wireshark, Nikto** | Use Kali terminal for WHOIS, nslookup, Nmap, theHarvester, Netdiscover, Sublist3r and Nikto; Armitage and Wireshark are graphical applications, with optional CLI companion tools such as TShark. |
| **Browser** | **Robtex, Shodan, Google Dorking** | Use the websites directly. In this guide Shodan is browser-based—no CLI or API key is required for the basic practical—and Robtex uses web lookups. |

**Important tool/platform clarifications**
- **Maltego:** in your practical workflow, open the Windows desktop application and use its graph/transform interface; do not start by typing `maltego` in Kali.
- **Shodan:** use `https://www.shodan.io` in the browser for the practical described here; its CLI is optional and not part of your assigned setup.
- **Nessus:** although the browser is used to interact with it, the Nessus scanner service is installed on Windows and normally accessed at `https://localhost:8834`.
- **Nikto:** your platform list omitted Nikto, but it was in the original 16-tool guide and lab material, so it is retained under Kali Linux. Check your instructor's final tool list if this model practical has a different scope.

## How to answer a practical question for full credit

For each scenario, prepare to state: **(1) aim, (2) platform and prerequisites, (3) command/menu path, (4) meaning of the key output fields, (5) security implication, (6) one limitation, (7) a sensible remediation or next step, and (8) result based on actual evidence.** A command typed without interpreting output is usually an incomplete explanation. Never make up a successful result if the tool fails or returns no data.

---

# 📘 EXPERIMENT 1: Sysmon, theHarvester, NetDiscover & Google Dorking

---

## 1a. Sysmon – Install, Configure & Analyze Event Logs (Windows)

### What is Sysmon?
**System Monitor (Sysmon)** is a Windows system service and device driver from Microsoft Sysinternals that logs detailed system activity to the Windows Event Log — including process creations, network connections, file changes, and more. It is widely used for **threat detection and incident response**.

### Step 1: Download Sysmon

Go to the official Microsoft Sysinternals page:
```
https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
```
Download and extract the ZIP file. You'll get:
- `Sysmon.exe` (32-bit)
- `Sysmon64.exe` (64-bit)

### Step 2: Download a Configuration File

Use the popular **SwiftOnSecurity Sysmon config**:
```
https://github.com/SwiftOnSecurity/sysmon-config
```
Download the file `sysmonconfig-export.xml`.

### Step 3: Install Sysmon with Configuration

Open **Command Prompt as Administrator** and run:

```cmd
cd C:\Users\YourUser\Downloads\Sysmon

Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

**Illustrative output (version text varies):**
```
System Monitor v15.0 - System activity monitor
By Mark Russinovich and Thomas Garnier
Copyright (C) 2014-2024 Microsoft Corporation
Using libxml2. libxml2 is Copyright (C) 1998-2012 Daniel Veillard. All Rights Reserved.
Sysmon installed.
SysmonDrv installed.
Starting SysmonDrv.
SysmonDrv started.
Starting Sysmon..
Sysmon started.
```

### Step 4: Verify Installation

```cmd
sc query Sysmon64
```

If the installed service uses a different name/version, inspect it with `sc query type= service state= all | findstr /I Sysmon` or open `services.msc`; use the service name shown on your machine.

**Expected Output:**
```
SERVICE_NAME: Sysmon64
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
        WIN32_EXIT_CODE    : 0
        SERVICE_EXIT_CODE  : 0
```

### Step 5: View Sysmon Logs in Event Viewer

1. Press `Win + R` → type `eventvwr.msc` → press Enter
2. Navigate to: **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**

### Step 6: Analyze Key Event IDs

| Event ID | Description | Security Relevance |
|----------|------------|-------------------|
| **1** | **Process Creation** | Detects malware execution, suspicious processes |
| **3** | **Network Connection** | Detects C2 callbacks, data exfiltration |
| **5** | Process Terminated | Tracks process lifecycle |
| **7** | Image Loaded (DLL) | Detects DLL injection attacks |
| **8** | CreateRemoteThread | Detects code injection |
| **10** | Process Access | Detects credential dumping (e.g., Mimikatz) |
| **11** | File Created | Detects malware file drops |
| **12/13/14** | Registry Events | Detects persistence mechanisms |
| **22** | DNS Query | Detects suspicious DNS lookups |

### Step 7: Example – Analyzing Process Creation (Event ID 1)

In Event Viewer, click on an **Event ID 1** entry. You will see:

```
Log Name:      Microsoft-Windows-Sysmon/Operational
Source:        Microsoft-Windows-Sysmon
Event ID:      1
Task Category: Process Create (rule: ProcessCreate)
Description:
  RuleName:        technique_id=T1059
  UtcTime:         2026-08-25 12:30:45.123
  ProcessGuid:     {abc12345-...}
  ProcessId:       4528
  Image:           C:\Windows\System32\cmd.exe
  CommandLine:     cmd.exe /c ipconfig
  CurrentDirectory: C:\Users\Student\
  User:            STUDENT-PC\Student
  ParentImage:     C:\Windows\explorer.exe
  ParentCommandLine: C:\Windows\explorer.exe
```

### Step 8: Example – Analyzing Network Connection (Event ID 3)

```
Event ID:      3
Task Category: Network connection detected (rule: NetworkConnect)
Description:
  RuleName:        technique_id=T1071
  UtcTime:         2026-08-25 12:31:10.456
  ProcessId:       7892
  Image:           C:\Program Files\Mozilla Firefox\firefox.exe
  User:            STUDENT-PC\Student
  Protocol:        tcp
  SourceIp:        192.168.1.100
  SourcePort:      52341
  DestinationIp:   142.250.195.68
  DestinationPort: 443
  DestinationHostname: www.google.com
```

### Step 9: Filter Specific Events using PowerShell

```powershell
# Get last 10 Process Creation events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" `
  -FilterXPath "*[System[EventID=1]]" -MaxEvents 10 | Format-List

# Get last 10 Network Connection events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" `
  -FilterXPath "*[System[EventID=3]]" -MaxEvents 10 | Format-List

# Get DNS Query events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" `
  -FilterXPath "*[System[EventID=22]]" -MaxEvents 10 | Format-List
```

### Sysmon Useful Commands Reference

| Command | Purpose |
|---------|---------|
| `Sysmon64.exe -accepteula -i config.xml` | Install with config |
| `Sysmon64.exe -c config.xml` | Update configuration |
| `Sysmon64.exe -u` | Uninstall Sysmon |
| `sc query Sysmon64` | Check service status |
| `eventvwr.msc` | Open Event Viewer |

> [!TIP]
> **Key Exam Points:**
> - Sysmon is a **host-based monitoring tool** (not network-based)
> - It logs to **Windows Event Log** under `Microsoft-Windows-Sysmon/Operational`
> - Event ID 1 = Process Creation, Event ID 3 = Network Connection
> - Configuration XML controls **what gets logged**
> - It does NOT block threats — it only **monitors and logs**

### Extra Sysmon viva checkpoints
- **Why is an XML configuration needed?** It controls which event types and event details are included; an overly broad config can create excessive logs, while a narrow config can miss activity.
- **Where are Sysmon events stored?** In Event Viewer under `Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`.
- **What do Event IDs 1, 3 and 22 mean?** Process creation, network connection and DNS query, respectively—provided the configuration enables those event types.
- **Is Sysmon an antivirus or firewall?** No. It records telemetry for monitoring/investigation; another product or response process is needed to block activity.
- **What should you correlate in a suspicious event?** Timestamp, image/process path, command line, parent process, user, destination IP/port and related events. One event alone may be benign.

---

## 1b. theHarvester – Passive Reconnaissance

### What is theHarvester?
**theHarvester** is an OSINT (Open Source Intelligence) tool used for **passive reconnaissance**. It collects email addresses, subdomains, hostnames, open ports, and banners from public sources like search engines and APIs — **without directly touching the target**.

### Step 1: Open Terminal in Kali Linux

theHarvester may be present in Kali already. Verify it before installing:

```bash
command -v theHarvester || command -v theharvester
theHarvester -h 2>/dev/null || theharvester -h
```

If missing, try `sudo apt update && sudo apt install theharvester`. Backend names and available data sources vary by version; use the help output to choose a source supported by the installed copy.

### Step 2: Basic Syntax

```bash
theHarvester -d <domain> -l <limit> -b <source>
```

| Flag | Meaning |
|------|---------|
| `-d` | Target domain |
| `-l` | Limit of results to retrieve |
| `-b` | Data source (google, bing, linkedin, all, etc.) |

### Step 3: Run theHarvester Against an Authorized Domain

```bash
theHarvester -d example.com -l 200 -b crtsh
```

`crtsh` is an example backend where supported. If it is not listed by `theHarvester -h`, choose a backend actually shown by your version. Do not assume every historical backend (for example, Google or LinkedIn) is still available without API credentials.

**Illustrative output only:**
```
*******************************************************************
*  _   _                                            _             *
* | |_| |__   ___    /\  /\__ _ _ ____   _____  ___| |_ ___ _ __ *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ \/ __| __/ _ \ '__|*
* | |_| | | |  __/ / __  / (_| | |   \ V /  __/\__ \ ||  __/ |   *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___||___/\__\___|_|   *
*                                                                 *
* theHarvester 4.6.0                                              *
*******************************************************************

[*] Target: example.com

[*] Searching Google.

[*] Emails found: 3
----------------------
admin@example.com
info@example.com
support@example.com

[*] Hosts found: 5
---------------------
www.example.com: 93.184.216.34
mail.example.com: 93.184.216.35
blog.example.com: 93.184.216.36
api.example.com: 93.184.216.37
dev.example.com: 93.184.216.38

[*] Subdomains found: 5
-------------------------
www.example.com
mail.example.com
blog.example.com
api.example.com
dev.example.com
```

### Step 4: Use Multiple Data Sources

```bash
theHarvester -d example.com -l 500 -b all
```

Use `-b all` only if `all` is supported in your installed version. Otherwise select one or more listed backends according to the syntax shown by `theHarvester -h`. Data-source availability and API-key requirements change over time.

### Step 5: Save Results to an HTML File

```bash
theHarvester -d example.com -l 200 -b <supported-backend> -f results_example
```

Depending on the installed version, the output files and formats may differ; check the completion message and the `-h` help text for the exact behavior.

> [!TIP]
> **Key Exam Points:**
> - theHarvester is a **passive** reconnaissance tool (doesn't directly interact with the target)
> - It gathers: **emails, subdomains, hostnames, IPs, banners**
> - Data sources depend on the current version and may require API credentials.
> - Use `-b all` only if the installed version explicitly supports it; otherwise choose a listed backend.
> - Use `-f` flag to save output to a file

### Extra theHarvester viva checkpoints
- **What is the difference between a domain and a subdomain?** `example.com` is a domain; `mail.example.com` or `www.example.com` is a subdomain/hostname beneath it.
- **Why might zero emails be returned?** The chosen backend may not have results, access may require an API key, rate limits may apply, or the domain may have little indexed data. Zero results do not prove no emails exist.
- **What does `-d`, `-l`, `-b`, and `-f` do?** Domain, result limit, backend/source, and output-file prefix, respectively (check the installed version's help).
- **Is every result verified or current?** No. OSINT data may be old, incomplete, or associated incorrectly; independently validate important observations.
- **What is passive reconnaissance?** Collecting information from public/third-party sources without directly probing the target for the data in question.

---

## 1c. NetDiscover – Network Host Discovery

### What is NetDiscover?
**NetDiscover** is an active/passive ARP reconnaissance tool used to **discover hosts on a local network**. It sends ARP requests and listens for responses to identify live hosts along with their **IP addresses** and **MAC addresses**.

### Step 1: Check Your Network Interface

```bash
ifconfig
```
or
```bash
ip addr show
```

Note your interface name (e.g., `eth0`, `wlan0`) and your subnet (e.g., `192.168.1.0/24`).

### Step 2: Run NetDiscover (Active Mode)

```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

| Flag | Meaning |
|------|---------|
| `-i` | Network interface to use |
| `-r` | Range/subnet to scan |

**Expected Output:**
```
 Currently scanning: 192.168.1.0/24   |   Screen View: Unique Hosts

 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.1.1     aa:bb:cc:dd:ee:01      1      60  TP-LINK TECHNOLOGIES
 192.168.1.100   aa:bb:cc:dd:ee:02      1      60  Dell Inc.
 192.168.1.101   aa:bb:cc:dd:ee:03      1      60  Apple, Inc.
 192.168.1.105   aa:bb:cc:dd:ee:04      1      60  VMware, Inc.
```

### Step 3: Run NetDiscover (Passive Mode)

```bash
sudo netdiscover -i eth0 -p
```

The `-p` flag enables **passive mode** — it only **listens** for ARP traffic without sending any packets (stealthier).

### Step 4: Scan the verified lab subnet

```bash
sudo netdiscover -i eth0 -r <authorized-subnet>
```

Replace `<authorized-subnet>` with the range shown by `ip -br addr` and your lab network design, for example `192.168.56.0/24` **only if that is actually your assigned host-only subnet**. Avoid scanning a broad range just because it appears in a sample. A VirtualBox NAT adapter may not expose all peer VMs to ARP discovery; use the network mode required by your lab.


### Understanding the Output

| Column | Meaning |
|--------|---------|
| **IP** | IP address of the discovered host |
| **At MAC Address** | MAC (hardware) address of the host |
| **Count** | Number of ARP packets captured from this host |
| **Len** | Length of ARP packets |
| **MAC Vendor** | Manufacturer identified from MAC OUI |

> [!TIP]
> **Key Exam Points:**
> - NetDiscover uses **ARP protocol** for host discovery (Layer 2)
> - Works only on **local networks** (same broadcast domain)
> - Active mode (`-r`) sends ARP requests; Passive mode (`-p`) only listens
> - Identifies: **IP address, MAC address, and vendor/manufacturer**
> - Must be run with **sudo** (root privileges)

### Extra Netdiscover viva checkpoints
- **What is ARP?** Address Resolution Protocol maps an IPv4 address to a link-layer (MAC) address on the local network segment.
- **Why does Netdiscover not normally find remote Internet hosts?** ARP broadcasts are local-link mechanisms and routers do not forward them across routed networks.
- **What is the MAC vendor column based on?** The organizationally unique identifier (OUI) portion of a MAC address; the lookup may be unknown, randomized, or inaccurate.
- **What is the difference between active and passive mode?** Active mode sends ARP probes; passive mode listens for existing ARP traffic.
- **Why can a host be missing from results?** It may be on another VLAN/subnet, powered off, isolated by the VM network mode, or not have generated visible ARP traffic.

---

## 1d. Google Dorking – Passive Reconnaissance with Advanced Search

### What is Google Dorking?
**Google Dorking** (also called Google Hacking) uses **advanced search operators** in Google to find publicly available but sensitive information about a target — such as login pages, exposed files, directory listings, and more.

### Key Google Dork Operators

**Compatibility note:** Search operators evolve. Operators such as `cache:` and `link:` shown in some older notes may be deprecated or unsupported by Google; prioritize currently working operators such as `site:`, `filetype:`, `intitle:`, `inurl:`, and `intext:` and describe the results actually returned.

| Operator | Description | Example |
|----------|-------------|---------|
| `site:` | Restrict results to a domain | `site:example.com` |
| `intitle:` | Search in page title | `intitle:"index of"` |
| `inurl:` | Search in URL | `inurl:admin` |
| `filetype:` | Search for file types | `filetype:pdf` |
| `intext:` | Search in page body | `intext:"password"` |
| `ext:` | File extension | `ext:sql` |
| `-` (minus) | Exclude results | `site:example.com -www` |

### Practical Google Dork Queries for Exam

**1. Find subdomains of a target:**
```
site:example.com -www
```
This searches for indexed pages under the domain while excluding results containing `www`; it does **not** guarantee a complete subdomain list.

**2. Find login/admin pages:**
```
site:example.com inurl:login
site:example.com inurl:admin
site:example.com intitle:"admin panel"
```

**3. Find exposed documents:**
```
site:example.com filetype:pdf
site:example.com filetype:doc OR filetype:docx OR filetype:xls
site:example.com filetype:sql
```

**4. Find directory listings:**
```
site:example.com intitle:"index of"
site:example.com intitle:"index of" "parent directory"
```

**5. Find sensitive information:**
```
site:example.com intext:"username" intext:"password"
site:example.com filetype:env
site:example.com filetype:log
```

**6. Find email addresses:**
```
site:example.com intext:"@example.com"
```

**7. Find exposed configuration files:**
```
site:example.com filetype:xml
site:example.com filetype:conf
site:example.com filetype:cfg
```

### Step-by-Step: How to Perform Google Dorking in Exam

1. Open your browser and go to `https://www.google.com`.
2. Scope the query to the assigned/authorized domain, for example `site:example.com filetype:pdf`.
3. Review result titles, URLs, snippets and dates; record only what is relevant to the question.
4. Describe the possible exposure and recommend an owner-side fix. Do not attempt to log in, bypass access controls, or download/use sensitive records merely because they are indexed.

Google changes or removes some search operators over time, so focus on widely used operators such as `site:`, `filetype:`, `intitle:`, `inurl:`, and `intext:` rather than relying on legacy operators.

### Example Workflow:

```
Step 1: site:targetdomain.com
        → Shows all indexed pages

Step 2: site:targetdomain.com -www
        → Shows subdomains (blog., mail., dev., etc.)

Step 3: site:targetdomain.com filetype:pdf
        → Shows publicly available PDF documents

Step 4: site:targetdomain.com intitle:"index of"
        → Shows directory listings (potential data exposure)

Step 5: site:targetdomain.com inurl:login
        → Shows login pages
```

> [!TIP]
> **Key Exam Points:**
> - Google Dorking is generally treated as **passive reconnaissance** because it searches the search engine's index rather than directly scanning the target; visiting results may generate normal web requests.
> - No tools needed — just use Google Search
> - Key operators: `site:`, `filetype:`, `intitle:`, `inurl:`, `intext:`
> - **Google Hacking Database (GHDB)** at `exploit-db.com/google-hacking-database` has 1000s of dorks
> - Search operators and indexed results can change; a query returning no results is not proof the content does not exist.
> - Use on authorized domains for an assessment, and do not access or redistribute exposed personal/confidential information.

### Extra Google Dorking viva checkpoints
- **Is Google Dorking a separate program?** No. It is a search technique using operators in a search engine.
- **What does `site:` do?** Restricts results to a specified domain/site.
- **What does `filetype:` do?** Limits results to a specified document/file type when indexed.
- **What is the risk of an exposed backup/config file?** It may disclose implementation details, credentials, or internal information; the owner should remove it from public access and rotate any exposed secrets.
- **Does an indexed result always mean the file is reachable now?** No. Search indexes can be stale, and access restrictions may have changed.

---
---

# 📗 EXPERIMENT 2: FOCA, WHOIS, nsLookUp & Sublist3r

---

## 2a. FOCA – Metadata Extraction from Documents

### What is FOCA?
**FOCA (Fingerprinting Organizations with Collected Archives)** is a Windows-based tool used to extract **metadata** from publicly available documents (PDF, DOCX, XLSX, PPTX, etc.) of a target organization. Metadata can reveal usernames, software versions, internal paths, email addresses, and printer names.

### Step 1: Download and Install FOCA

Download from: `https://github.com/ElevenPaths/FOCA`
- FOCA runs on **Windows**
- Requires **.NET Framework** and **SQL Server Express** (LocalDB)

### Step 2: Create a New Project

1. Open FOCA
2. Click **Project → New Project**
3. Enter:
   - **Project Name**: `TargetRecon`
   - **Domain**: `example.com`
   - **Output Folder**: Choose a location to save results

### Step 3: Search for Documents

1. In the left panel, you'll see the domain `example.com`
2. Click on **Network → Search Engines**
3. Select search engines: **Google, Bing**
4. Select file types to search for: **PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX**
5. Click **Search All** button

FOCA will search Google and Bing for documents from the target domain.

### Step 4: Download Documents

1. After search completes, right-click on the found documents
2. Select **Download All** or download individually
3. Wait for downloads to complete

### Step 5: Extract Metadata

1. Right-click on downloaded documents
2. Select **Extract Metadata** → **Extract All Metadata**
3. FOCA will analyze each document and extract:

**Metadata Information Found:**

| Metadata Field | Example Value | Security Relevance |
|---------------|---------------|-------------------|
| **Author/Creator** | John.Smith | Reveals employee names → phishing targets |
| **Software** | Microsoft Office 2019 | Reveals software versions → known vulnerabilities |
| **Operating System** | Windows 10 Pro | Reveals OS → targeted exploits |
| **Internal Paths** | `C:\Users\jsmith\Documents\` | Reveals internal folder structure |
| **Printer Name** | `HP-LaserJet-4F-Finance` | Reveals internal infrastructure |
| **Email Address** | `jsmith@example.com` | Reveals email format & valid addresses |
| **Creation Date** | 2024-01-15 | Timeline of organizational activity |
| **Last Modified By** | Admin_IT | Reveals admin accounts |

### Step 6: Analyze the Metadata Tree

In FOCA's left panel, expand the metadata tree:
```
example.com
├── Users Found
│   ├── John.Smith
│   ├── Admin_IT
│   └── jane.doe
├── Software Detected
│   ├── Microsoft Office 2019
│   └── Adobe Acrobat Pro DC
├── Operating Systems
│   ├── Windows 10
│   └── Windows Server 2019
├── Printers
│   └── HP-LaserJet-4F-Finance
├── Folders/Paths
│   ├── C:\Users\jsmith\Documents\Reports
│   └── \\fileserver\shared\HR
└── Emails
    ├── jsmith@example.com
    └── it-admin@example.com
```

> [!TIP]
> **Key Exam Points:**
> - FOCA extracts **metadata** from public documents (PDF, DOC, XLS, PPT)
> - Reveals: **usernames, software, OS, email addresses, internal paths, printers**
> - It searches Google/Bing for documents → downloads them → extracts metadata
> - This is **passive** reconnaissance (documents are already public)
> - FOCA is a **Windows desktop GUI** in this practical setup.
> - Metadata varies by file format and may be absent, anonymized, or misleading.

### Extra FOCA viva checkpoints
- **What is metadata leakage?** Unintended disclosure of information stored in a document's properties or embedded data.
- **What is FOCA's basic workflow?** Create a project → search for public documents → download selected documents → extract/analyze metadata → document findings.
- **Why can metadata be useful to defenders?** It reveals information that an organization may choose to remove or standardize before publishing documents.
- **What remediation is appropriate?** Review document metadata before release, remove unnecessary fields, and apply a publication process; removing metadata does not replace access controls.
- **Why might FOCA not connect to the database?** SQL Server may be absent/not running, the instance name or authentication may be wrong, or the installed dependencies may be incompatible.

---

## 2b. WHOIS Reconnaissance

### What is WHOIS?
**WHOIS** is a query protocol used to look up **domain registration information**. It reveals who registered the domain, when, contact details, name servers, and registrar information.

### Step 1: Verify and run WHOIS in Kali Linux

```bash
command -v whois
whois example.com
```

If the command is missing, install it with `sudo apt update && sudo apt install whois`. Use the assigned domain if your question names one; `example.com` is a reserved example domain used for illustration.

**Expected Output:**
```
   Domain Name: EXAMPLE.COM
   Registry Domain ID: 2336799_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.iana.org
   Registrar URL: http://www.iana.org
   Updated Date: 2024-08-14T07:01:44Z
   Creation Date: 1995-08-14T04:00:00Z
   Registry Expiry Date: 2025-08-13T04:00:00Z
   Registrar: RESERVED-Internet Assigned Numbers Authority
   Registrar IANA ID: 376
   Registrar Abuse Contact Email: abuse@iana.org
   Registrar Abuse Contact Phone: +1.3108239358

   Domain Status: clientDeleteProhibited
   Domain Status: clientTransferProhibited
   Domain Status: clientUpdateProhibited

   Name Server: A.IANA-SERVERS.NET
   Name Server: B.IANA-SERVERS.NET

   Registrant Name: REDACTED FOR PRIVACY
   Registrant Organization: Internet Assigned Numbers Authority
   Registrant Street: REDACTED FOR PRIVACY
   Registrant City: Los Angeles
   Registrant State/Province: CA
   Registrant Country: US
   Registrant Phone: +1.3108239358
   Registrant Email: domaincontact@icann.org
```

### Step 2: WHOIS for an IP Address

```bash
whois 93.184.216.34
```

### Step 3: Online WHOIS Lookup (Alternative)

You can also use web-based WHOIS tools:
- `https://whois.domaintools.com`
- `https://who.is`
- `https://www.whois.com/whois/`

### Key Information Gathered from WHOIS:

| Information | Description |
|------------|-------------|
| **Domain Name** | The registered domain |
| **Registrar** | Company where domain was registered (GoDaddy, Namecheap, etc.) |
| **Creation Date** | When the domain was first registered |
| **Expiry Date** | When the registration expires |
| **Name Servers** | DNS servers handling the domain |
| **Registrant Info** | Owner's name, organization, address, phone, email |
| **Admin Contact** | Administrative contact details |
| **Tech Contact** | Technical contact details |
| **Domain Status** | Status flags (locked, active, etc.) |

> [!TIP]
> **Key Exam Points:**
> - WHOIS is a **passive** reconnaissance technique
> - Reveals: **registrar, name servers, registration dates, contact info**
> - Many domains use **WHOIS privacy protection** (REDACTED)
> - Command: `whois <domain>`
> - Works for domains and many IP allocation lookups, for example `whois 93.184.216.34`.
> - Privacy services often redact personal contact details; missing details are not a tool failure.

### Extra WHOIS viva checkpoints
- **What is a registrar?** The organization through which a domain registration is managed.
- **What are creation and expiry dates?** Registration lifecycle dates shown by the relevant registry/registrar; date semantics can vary.
- **What does a name server indicate?** The server delegated to provide authoritative DNS for the domain.
- **Does WHOIS always identify the actual individual owner?** No. Privacy/proxy services and registry policy may mask or omit the registrant.
- **WHOIS versus DNS lookup?** WHOIS provides registration/allocation records; DNS lookup resolves names and queries DNS resource records.

---

## 2c. nsLookup – DNS Reconnaissance

### What is nsLookup?
**nsLookup** (Name Server Lookup) is a command-line tool used to **query DNS servers** to find IP addresses, mail servers, name servers, and other DNS records for a domain.

### Step 1: Basic Domain Lookup

```bash
nslookup example.com
```

**Expected Output:**
```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   example.com
Address: 93.184.216.34
Name:   example.com
Address: 2606:2800:220:1:248:1893:25c8:1946
```

### Step 2: Find Mail Servers (MX Records)

```bash
nslookup -type=mx example.com
```

**Expected Output:**
```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
example.com     mail exchanger = 10 mail.example.com.
example.com     mail exchanger = 20 mail2.example.com.
```

### Step 3: Find Name Servers (NS Records)

```bash
nslookup -type=ns example.com
```

**Expected Output:**
```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
example.com     nameserver = a.iana-servers.net.
example.com     nameserver = b.iana-servers.net.
```

### Step 4: Understand ANY queries (optional)

```bash
nslookup -type=any example.com
```

An `ANY` query is not a reliable way to retrieve every DNS record. Resolvers and authoritative servers may limit or synthesize responses. For a practical answer, query the specific record types required (A, AAAA, MX, NS, SOA, TXT, or CNAME).

### Step 5: Find SOA (Start of Authority) Record

```bash
nslookup -type=soa example.com
```

**Expected Output:**
```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
example.com
        origin = ns.icann.org
        mail addr = noc.dns.icann.org
        serial = 2024081401
        refresh = 7200
        retry = 3600
        expire = 1209600
        minimum = 3600
```

### Step 6: Find Text Records (TXT)

```bash
nslookup -type=txt example.com
```

This can reveal **SPF records, DKIM keys, domain verification strings**.

### Step 7: Query a Specific DNS Server

```bash
nslookup example.com 8.8.8.8
```

### Step 8: Interactive Mode

```bash
nslookup
> set type=mx
> example.com
> set type=ns
> example.com
> exit
```

### DNS Record Types Summary:

| Record Type | Purpose | Command |
|------------|---------|---------|
| **A** | IPv4 Address | `nslookup example.com` |
| **AAAA** | IPv6 Address | `nslookup -type=aaaa example.com` |
| **MX** | Mail Servers | `nslookup -type=mx example.com` |
| **NS** | Name Servers | `nslookup -type=ns example.com` |
| **SOA** | Start of Authority | `nslookup -type=soa example.com` |
| **TXT** | Text Records (SPF, DKIM) | `nslookup -type=txt example.com` |
| **CNAME** | Canonical Name (Alias) | `nslookup -type=cname www.example.com` |
| **PTR** | Reverse DNS | `nslookup 93.184.216.34` (resolver may perform a reverse lookup) |

> [!TIP]
> **Key Exam Points:**
> - nsLookup queries **DNS servers** for domain information
> - Key record types: **A** (IP), **MX** (mail), **NS** (nameserver), **SOA**, **TXT**
> - Use `-type=<record>` to query specific record types
> - It is a **passive** reconnaissance technique
> - Your assigned workflow uses Kali terminal; `nslookup` is also available on Windows.
> - If the resolver returns `NXDOMAIN`, the queried name may not exist in DNS; other failures can reflect timeouts, DNS server issues, or filtering.

### Extra nslookup viva checkpoints
- **What is a DNS resolver?** A server that receives DNS queries and returns answers or resolves them via other DNS servers.
- **Authoritative vs non-authoritative answer?** An authoritative answer comes from a server authoritative for the zone; a non-authoritative answer is typically returned by a recursive resolver/cache.
- **A vs AAAA?** A maps a name to IPv4; AAAA maps a name to IPv6.
- **MX vs NS?** MX records specify mail exchangers; NS records specify authoritative name servers.
- **What is TTL?** Time to Live, the period a DNS record may be cached under its DNS TTL value.
- **Why do DNS results change?** Records may be updated, cached until TTL expiry, geographically varied, or provided by load-balanced/CDN infrastructure.

---

## 2d. Sublist3r – Passive Subdomain Enumeration

### What is Sublist3r?
**Sublist3r** is a Python-based tool designed to enumerate **subdomains** of a target domain using passive sources like search engines (Google, Bing, Yahoo, Baidu), VirusTotal, ThreatCrowd, DNSdumpster, and more.

### Step 1: Verify or install Sublist3r

Check whether it is already available:

```bash
command -v sublist3r
sublist3r -h
```

On Kali, prefer the maintained distribution package when available:

```bash
sudo apt update
sudo apt install sublist3r
```

If the package is unavailable and the instructor specifically requires a source checkout, follow the repository's current README and use an isolated Python virtual environment. Kali may block global `pip` installs into its system Python; do not force a global install just to bypass that protection.

### Step 2: Basic Subdomain Enumeration

```bash
sublist3r -d example.com
```

**Expected Output:**
```
                 ____        _     _ _     _   _____
                / ___| _   _| |__ | (_)___| |_|___ / _ __
                \___ \| | | | '_ \| | / __| __| |_ \| '__|
                 ___) | |_| | |_) | | \__ \ |_ ___) | |
                |____/ \__,_|_.__/|_|_|___/\__|____/|_|

                # Coded By Ahmed Aboul-Ela - @ababorahma

[-] Enumerating subdomains now for example.com
[-] Searching now in Google..
[-] Searching now in Bing..
[-] Searching now in Yahoo..
[-] Searching now in Baidu..
[-] Searching now in DNSdumpster..
[-] Searching now in VirusTotal..
[-] Searching now in ThreatCrowd..
[-] Searching now in Netcraft..

[-] Total Unique Subdomains Found: 8
www.example.com
mail.example.com
blog.example.com
dev.example.com
api.example.com
staging.example.com
portal.example.com
vpn.example.com
```

### Step 3: Save Output to a File

```bash
sublist3r -d example.com -o subdomains.txt
```

### Step 4: Use Specific Search Engines

```bash
sublist3r -d example.com -e <supported-engines-comma-separated>
```

### Step 5: Enumerate with Port Scanning

```bash
sublist3r -d example.com -p 80,443,8080
```

This also checks if common ports are open on discovered subdomains.

### Sublist3r Flags Reference:

| Flag | Meaning |
|------|---------|
| `-d` | Target domain |
| `-o` | Output file |
| `-e` | Specific search engines |
| `-b` | Enable brute force (active, not passive) |
| `-t` | Number of threads |
| `-p` | Scan found subdomains for open ports |

> [!TIP]
> **Key Exam Points:**
> - Sublist3r is a **passive** subdomain enumeration tool
> - Uses search engines and public databases as data sources
> - Without `-b` flag, it is entirely passive
> - Finds subdomains that may not be found by simple Google dorking
> - Use `-o` to save results to a file.
> - Search engines/APIs change, and subdomain lists are not guaranteed complete. Check whether discovered hosts still resolve and are in scope before further testing.

### Extra Sublist3r viva checkpoints
- **What is subdomain enumeration?** Identifying hostnames beneath a domain, such as `mail.example.com` or `api.example.com`.
- **Why does it return duplicates?** Different sources may report the same name; tools commonly deduplicate the combined list.
- **What does `-e` do?** Selects the supported search engines/backends; valid values depend on the installed tool version.
- **What do `-b` and `-p` do?** `-b` enables brute-force enumeration; `-p` requests port checks against discovered subdomains. These can create active traffic.
- **Why can it miss subdomains?** Private/unindexed DNS names, source limits, API limits, or recent changes may not appear.

---
---

# 📕 EXPERIMENT 3: Maltego, Shodan, Nikto & NMAP

---

## 3a. Maltego – OSINT Reconnaissance & Visualization

### What is Maltego?
**Maltego** is a powerful OSINT (Open Source Intelligence) and graphical link analysis tool. It performs automated reconnaissance and **visualizes relationships** among domains, email addresses, IP addresses, people, social networks, and infrastructure in an interactive graph.

### Step 1: Open Maltego on Windows (your assigned platform)

Maltego is a **desktop GUI application** in your practical workflow—do not start with a terminal command. Download it only from the official page: <https://www.maltego.com/downloads/>. Install it on Windows, register/sign in, and select the Community Edition or lab-licensed edition available to you. Exact account, data-source and transform limits depend on the current edition.

### Step 2: Register and Log In

1. First launch → Register for a **Maltego Community Edition (CE)** account
2. Enter your email and create a password
3. Log in with your credentials

### Step 3: Create a New Graph

1. Click **New Graph** (or `Ctrl + T`)
2. You'll see a blank canvas (the graph workspace)

### Step 4: Add the Target Entity

1. From the **Entity Palette** (left panel), drag a **Domain** entity onto the canvas
2. Double-click the entity and type your target domain: `example.com`

### Step 5: Run Transforms

**Transforms** are Maltego's automated queries that gather information.

Right-click on the domain entity → **Run Transform** → Choose from:

| Transform | What it Does |
|-----------|-------------|
| **To DNS Name** | Finds DNS records (A, MX, NS) |
| **To Email Addresses** | Finds email addresses associated with the domain |
| **To IP Address [DNS]** | Resolves domain to IP addresses |
| **To MX Record** | Finds mail servers |
| **To NS Record** | Finds name servers |
| **To Website** | Finds associated websites |
| **To Subdomains** | Enumerates subdomains |
| **To WHOIS** | Gets WHOIS information |
| **All Transforms** | Runs all available transforms |

### Step 6: Expand the Graph

After running transforms, new entities appear on the graph. You can:
1. Right-click on any **new entity** → Run more transforms
2. For example: Right-click an **email address** → **To Person** → reveals the person's name
3. Right-click on an **IP address** → **To Netblock** → reveals the network range

### Step 7: Analyze the Visual Graph

The resulting graph shows relationships like:

```
                         example.com
                        /     |      \
                       /      |       \
             mail.example.com |   blog.example.com
                   |          |          |
              93.184.216.35   |     93.184.216.36
                              |
                       admin@example.com
                              |
                         John Smith
                              |
                        LinkedIn Profile
```

### Step 8: Graph Layout Options

Use the toolbar to change graph layouts:
- **Block Layout** – Organized hierarchical view
- **Circular Layout** – Radial arrangement
- **Organic Layout** – Natural graph flow
- **Hierarchical Layout** – Top-down tree view

### Step 9: Export Results

Go to **File → Export** → Choose format:
- Export as **PDF**, **CSV**, **Image (PNG/JPG)**, or **Excel**

> [!TIP]
> **Key Exam Points:**
> - Maltego performs **OSINT-based** reconnaissance with **visual graph output**
> - **Transforms** are automated data-gathering queries
> - Visualizes **relationships** between entities (domains, IPs, emails, people)
> - Community functionality, result limits, data-source access, and transform availability can change; do not memorize a fixed result limit unless it is shown by your installed edition.
> - Transforms may query third-party services; a transform is not guaranteed to return a result.

### Extra Maltego viva checkpoints
- **What is an entity?** A graph object representing an item such as a domain, IP address, email, person, or organization.
- **What is a transform?** An operation that queries a source or performs a lookup and creates related entities/links from the input entity.
- **Why use a graph instead of a flat list?** A graph makes relationships and pivot paths easier to visualize and explain.
- **What is a false relationship risk?** Shared hosting, recycled IPs, common names, stale records, or third-party enrichment can make two entities appear related without proving common ownership.
- **Is every transform passive?** Many OSINT transforms use public sources, but some transforms can perform direct lookups or interact with services; review the transform and authorization context.
- **What should be included in the result?** Screenshot/export of the graph, entity types, transforms used, key links, and a limitation statement.

---

## 3b. Shodan – Search Engine for Internet-Connected Devices

### What is Shodan?
**Shodan** is a search engine that indexes **Internet-connected devices** — servers, routers, webcams, IoT devices, SCADA systems, databases, etc. It shows open ports, running services, banners, software versions, and potential vulnerabilities.

### Method 1: Using Shodan Web Interface

1. Go to `https://www.shodan.io`
2. Create a **free account** (required for searching)
3. Log in

### Step 2: Basic Search

In the search bar, type:
```
hostname:example.com
```

Or search by IP:
```
93.184.216.34
```

**Expected Results Page Shows:**

```
┌──────────────────────────────────────────────┐
│  93.184.216.34                               │
│  Organization: Edgecast Inc.                 │
│  Country: United States                      │
│  City: Norwood                               │
│  ISP: Edgecast Inc.                          │
│                                              │
│  Open Ports:                                 │
│  ├── 80/tcp   - HTTP                         │
│  │   Server: ECS (sec/97A6)                  │
│  │   Content-Type: text/html                 │
│  ├── 443/tcp  - HTTPS                        │
│  │   SSL Version: TLSv1.2                    │
│  │   Certificate: example.com                │
│  │   Issuer: DigiCert                        │
│  └── 8080/tcp - HTTP (Alternate)             │
│      Server: Apache/2.4.41                   │
│                                              │
│  Technologies: ECS, TLS 1.2, HTTP/2          │
│  Vulnerabilities: CVE-2021-XXXXX             │
└──────────────────────────────────────────────┘
```

### Step 3: Useful Shodan Search Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `hostname:` | Search by hostname | `hostname:example.com` |
| `ip:` | Search specific IP | `ip:93.184.216.34` |
| `port:` | Filter by port | `port:22` |
| `org:` | Filter by organization | `org:"Google"` |
| `country:` | Filter by country | `country:IN` |
| `product:` | Filter by software | `product:Apache` |
| `os:` | Filter by OS | `os:"Windows Server 2019"` |
| `city:` | Filter by city | `city:"Chennai"` |
| `ssl:` | Search SSL certificates | `ssl:"example.com"` |
| `vuln:` | Search by CVE | `vuln:CVE-2021-44228` |
| `net:` | Search by CIDR | `net:192.168.1.0/24` |

### Keep the practical browser-based

For this exam setup, you do **not** need to install a Shodan package, configure an API key, or use a terminal. Search in the browser and inspect the results page. If your instructor asks about the CLI in viva, explain that it is an optional API-based interface separate from the browser workflow.

### Safe Shodan queries for the practical

Use only your own organization, instructor-provided domain, or an explicitly authorized public asset. Examples: use `hostname:your-authorized-domain.example` to narrow to a hostname, or `org:"Your Organization"` if your lab has provided the correct organization name. Do not search for or probe unrelated vulnerable devices.

```text
hostname:your-authorized-domain.example
org:"Your Organization"
port:443 hostname:your-authorized-domain.example
```

Replace these placeholders with authorized scope. A query may return no results, and that does not prove the asset is absent from the Internet.

> [!TIP]
> **Key Exam Points:**
> - Shodan indexes **publicly exposed** Internet-connected devices
> - Shows: **open ports, services, versions, banners, SSL certs, vulnerabilities**
> - Querying Shodan is **passive from the analyst's perspective**: you search Shodan's index rather than probing the listed target. The underlying data is collected by Shodan's own crawlers and may be stale.
> - Key filters: `hostname:`, `port:`, `org:`, `country:`, `product:`. Search access and filters can be subject to current account limits.
> - It is an **indexed view**, not a guaranteed real-time scan; check the reported last-seen/update time and validate authorized findings separately.

### Extra Shodan viva checkpoints
- **What is a banner?** Service response metadata that may reveal a product, version, protocol, or configuration.
- **Does Shodan itself prove that a port is open right now?** No. It reports indexed observations; validate current state only when authorized.
- **What does `port:443` mean?** Filter indexed results for services observed on port 443; it does not mean only HTTPS is guaranteed.
- **Shodan vs Nmap?** Shodan searches an existing Internet-wide index; Nmap actively probes a target/range from your own machine.
- **Why can a displayed CVE be misleading?** Product identification/version detection can be imperfect and systems may be patched/backported; validate applicability.

---

## 3c. Nikto – Web Server Vulnerability Scanner

### What is Nikto?
**Nikto** is an open-source **web server scanner** that checks for dangerous files, outdated server software, version-specific vulnerabilities, and server misconfigurations. It performs **active** scanning.

### Step 1: Verify installation and scan only a permitted lab web server

```bash
nikto -Version
command -v nikto
```

If missing, install from Kali repositories: `sudo apt update && sudo apt install nikto`. Use the instructor-assigned web server or intentionally vulnerable lab VM. Example syntax (replace the placeholder with that authorized target):

```bash
nikto -h http://<authorized-lab-web-server>
```

Nikto sends many HTTP requests; it is active and may be noisy in server logs. Do not scan a public site without explicit permission.

**Expected Output:**
```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          93.184.216.34
+ Target Hostname:    example.com
+ Target Port:        80
+ Start Time:         2026-08-25 12:00:00 (GMT5.5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present.
+ /: The X-Content-Type-Options header is not set. This could allow the
  user agent to render the content of the site in a different fashion
  to the MIME type.
+ /: Server may leak inodes via ETags, header found with file /,
  inode: 2aa6, size: 5c8a2c4c0d792, mtime: gzip
+ Apache/2.4.41 appears to be outdated (current is at least 2.4.59).
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS .
+ /icons/: Directory indexing found.
+ /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
+ /phpmyadmin/: phpMyAdmin directory found.
+ /server-status: Apache server-status page found (protected).
+ /test/: This might be interesting...
+ /backup/: Directory indexing found.
+ 8102 requests: 0 error(s) and 11 item(s) reported on remote host
+ End Time:           2026-08-25 12:05:30 (GMT5.5) (330 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

### Step 2: Scan HTTPS Server

```bash
nikto -h https://example.com -ssl
```

### Step 3: Scan Specific Port

```bash
nikto -h http://target-ip:8080
```

### Step 4: Save Output to File

```bash
# HTML report
nikto -h http://example.com -o report.html -Format htm

# Text report
nikto -h http://example.com -o report.txt -Format txt

# CSV report
nikto -h http://example.com -o report.csv -Format csv
```

### Step 5: Tuning Options (Specific Tests)

```bash
nikto -h http://example.com -Tuning 1234
```

| Tuning # | Test Type |
|----------|-----------|
| 0 | File Upload |
| 1 | Interesting File / Seen in logs |
| 2 | Misconfiguration / Default File |
| 3 | Information Disclosure |
| 4 | Injection (XSS/Script/HTML) |
| 5 | Remote File Retrieval – Inside Web Root |
| 6 | Denial of Service |
| 7 | Remote File Retrieval – Server Wide |
| 8 | Command Execution / Remote Shell |
| 9 | SQL Injection |

### What Nikto Identifies:

| Finding Type | Examples |
|-------------|----------|
| **Outdated Software** | Apache 2.4.41, PHP 7.2, OpenSSL 1.1.1 |
| **Dangerous Files** | `/phpinfo.php`, `/test.php`, backup files |
| **Default Files** | `/icons/README`, default index pages |
| **Misconfigurations** | Missing security headers, directory listing enabled |
| **Known Vulnerabilities** | CVE references for detected software |
| **Admin Pages** | `/admin/`, `/login.php`, `/phpmyadmin/` |
| **HTTP Methods** | PUT, DELETE enabled (security risk) |

### Nikto Flags Reference:

| Flag | Meaning |
|------|---------|
| `-h` | Target host (IP or URL) |
| `-p` | Target port |
| `-ssl` | Force SSL/HTTPS |
| `-o` | Output file |
| `-Format` | Output format (htm, csv, txt, xml) |
| `-Tuning` | Specific test categories |
| `-timeout` | Connection timeout (seconds) |
| `-id` | HTTP authentication (user:pass) |

> [!TIP]
> **Key Exam Points:**
> - Nikto is an **active** web server scanner (sends requests to the target)
> - Checks for: **vulnerabilities, misconfigs, outdated software, dangerous files**
> - Basic command: `nikto -h <target>`
> - Use `-ssl` for HTTPS targets
> - Use `-o` to save output reports
> - Not stealthy — **generates many log entries** on the target server. Findings should be validated, because a path/response match is not always a vulnerability.

### Extra Nikto viva checkpoints
- **Why is Nikto active?** It sends HTTP/HTTPS requests to the target web server to check known files, versions, and configuration issues.
- **What is a missing security header?** A response lacks a browser security policy header; risk depends on the application and threat model.
- **Does a Nikto finding prove exploitable vulnerability?** No. Review the exact request/response, server version, configuration, and remediation.
- **What does `-o` do?** Writes results to a file; `-Format` chooses a supported report format.
- **What are common mitigations?** Patch/upgrade the server, remove unnecessary files, disable directory listing, and configure appropriate headers and HTTP methods.

---

## 3d. Nmap – Active Network Reconnaissance

### What is Nmap?
**Nmap (Network Mapper)** is the most widely used **active reconnaissance** tool. It discovers live hosts, open ports, running services, service versions, and operating systems on a target network.

### Step 1: Basic Host Discovery (Ping Scan)

```bash
nmap -sn 192.168.1.0/24
```

**Illustrative output only; ports, versions and counts depend on the target:**
```
Starting Nmap 7.94SVN ( https://nmap.org )
Nmap scan report for 192.168.1.1
Host is up (0.0010s latency).
MAC Address: AA:BB:CC:DD:EE:01 (TP-Link Technologies)

Nmap scan report for 192.168.1.100
Host is up (0.0025s latency).
MAC Address: AA:BB:CC:DD:EE:02 (Dell)

Nmap scan report for 192.168.1.105
Host is up (0.0030s latency).
MAC Address: AA:BB:CC:DD:EE:03 (VMware)

Nmap done: 256 IP addresses (3 hosts up) scanned in 2.15 seconds
```

### Step 2: TCP SYN Scan (Default – Most Common)

```bash
sudo nmap -sS target-ip
```

**Expected Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org )
Nmap scan report for target-ip (192.168.1.105)
Host is up (0.0015s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
110/tcp  open  pop3
143/tcp  open  imap
443/tcp  open  https
3306/tcp open  mysql
```

### Step 3: Service Version Detection

```bash
nmap -sV target-ip
```

**Expected Output:**
```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.6
80/tcp   open  http        Apache httpd 2.4.52 ((Ubuntu))
443/tcp  open  ssl/http    Apache httpd 2.4.52 ((Ubuntu))
3306/tcp open  mysql       MySQL 8.0.35-0ubuntu0.22.04.1
```

### Step 4: Operating System Detection

```bash
sudo nmap -O target-ip
```

**Expected Output:**
```
...
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.4 - 5.15
Network Distance: 1 hop
```

### Step 5: Aggressive scan (only if the question/instructor asks)

```bash
sudo nmap -A <authorized-lab-target>
```

This combines service/version detection, OS detection, default scripts, and traceroute. It is noisier and more comprehensive than a basic port scan; do not treat it as the default for every target. Start with the narrowest scan that answers the question, and use aggressive scanning only within explicit authorization.

**Expected Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org )
Nmap scan report for target-ip (192.168.1.105)
Host is up (0.0015s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.6
| ssh-hostkey:
|   256 xx:xx:xx:xx (ECDSA)
|_  256 xx:xx:xx:xx (ED25519)
80/tcp   open  http     Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Welcome to Our Website
|_http-server-header: Apache/2.4.52 (Ubuntu)
443/tcp  open  ssl/http Apache httpd 2.4.52 ((Ubuntu))
| ssl-cert: Subject: commonName=example.com
| Not valid before: 2024-01-01T00:00:00
|_Not valid after:  2025-01-01T23:59:59
3306/tcp open  mysql    MySQL 8.0.35
| mysql-info:
|   Protocol: 10
|   Version: 8.0.35-0ubuntu0.22.04.1
|_  Salt: xxxxxxxxxxxxxxxxx

OS detection:
  Running: Linux 5.X
  OS details: Linux 5.4 - 5.15
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   1.50 ms 192.168.1.105

Nmap done: 1 IP address (1 host up) scanned in 25.67 seconds
```

### Step 6: Scan Specific Ports

```bash
# Specific ports
nmap -p 80,443,22,3306 target-ip

# Port range
nmap -p 1-1000 target-ip

# ALL 65535 ports
nmap -p- target-ip

# Top 100 most common ports
nmap --top-ports 100 target-ip
```

### Step 7: UDP Scan

```bash
sudo nmap -sU target-ip
```

### Step 8: Save Output

```bash
# Normal text output
nmap -A target-ip -oN scan_results.txt

# XML output
nmap -A target-ip -oX scan_results.xml

# All formats at once
nmap -A target-ip -oA scan_results
```

### Nmap Scan Types Summary:

| Scan Type | Flag | Description | Requires sudo? |
|-----------|------|-------------|---------------|
| **Ping Scan** | `-sn` | Host discovery only (no port scan) | No |
| **TCP SYN** | `-sS` | Half-open scan (stealthy, default) | **Yes** |
| **TCP Connect** | `-sT` | Full TCP connection scan | No |
| **UDP Scan** | `-sU` | Scans UDP ports | **Yes** |
| **Version Detect** | `-sV` | Detects service versions | No |
| **OS Detection** | `-O` | Detects operating system | **Yes** |
| **Aggressive** | `-A` | OS + Version + Scripts + Traceroute | **Yes** |
| **Script Scan** | `-sC` | Runs default NSE scripts | No |
| **All Ports** | `-p-` | Scans all 65535 ports | No |
| **Specific Ports** | `-p 22,80,443` | Scans only listed ports | No |

### Nmap Port States:

| State | Meaning |
|-------|---------|
| **open** | Port is accepting connections |
| **closed** | Port is reachable but no service running |
| **filtered** | Firewall is blocking the probe |
| **unfiltered** | Port is accessible but open/closed unknown |
| **open\|filtered** | Cannot determine if open or filtered |

> [!TIP]
> **Key Exam Points:**
> - Nmap is an **active** reconnaissance tool (sends packets to target)
> - Key flags: `-sS` (SYN), `-sV` (version), `-O` (OS), `-A` (aggressive)
> - `-A` combines multiple detection features and can be noisy; use it only when the practical question/instructor asks for comprehensive detection.
> - Discovers: **live hosts, open ports, services, versions, OS hints**; results are estimates, not guarantees.
> - Use `sudo` for SYN scan and OS detection (requires raw packets)
> - Use `-oN` or `-oA` to save results to files

### Extra Nmap viva checkpoints
- **What is port scanning?** Probing target ports to determine which appear open, closed, or filtered.
- **What is an SYN scan?** A TCP half-open scan that sends SYN probes and infers port state from responses without completing a normal TCP handshake in typical cases.
- **What does `-sV` do?** It probes services to estimate protocol/application and version.
- **What does `-O` do?** It attempts OS fingerprinting from network responses; accuracy depends on conditions and available responses.
- **What is the difference between host discovery and port scanning?** Host discovery looks for live/reachable hosts; port scanning tests selected ports on targets. `-sn` omits the normal port scan.
- **Why can a port show `filtered`?** A firewall or packet filtering prevents Nmap from distinguishing open from closed.
- **Can an open port be a vulnerability?** Not by itself. Assess the service, need for exposure, version, authentication, patching, and configuration.

---
---

# 📙 EXPERIMENT 4: Wireshark – Network Protocol and Packet Analysis

---

## 4a. What is Wireshark?

**Wireshark** is a network protocol analyzer (packet sniffer) used to capture, inspect, filter, and troubleshoot network traffic. It can capture traffic live from a network interface or open a previously saved capture file such as `.pcap` or `.pcapng`.

### Why is it used?
- Troubleshoot DNS resolution, TCP connections, and application connectivity.
- See the source and destination IP addresses, ports, protocols, timestamps, packet sizes, and protocol fields.
- Investigate retransmissions, TCP resets, failed handshakes, unusual DNS requests, and other symptoms.
- Collect evidence from a permitted lab capture for a report or incident investigation.

**Important:** Wireshark is an analyzer, not a firewall or vulnerability scanner. It does not automatically block traffic. On switched networks or virtual-machine NAT networks, it generally sees traffic visible to the selected interface—not automatically every packet from every computer.

### Step 1: Install or verify Wireshark on Kali Linux

Wireshark is already present in many Kali installations. Check before reinstalling:

```bash
wireshark --version
tshark --version
command -v wireshark
```

If it is missing, install it from Kali's repositories:

```bash
sudo apt update
sudo apt install wireshark tshark
```

If the installer asks whether users other than root may capture packets, select the option your lab policy requires. To configure non-root capture later, Kali/Debian systems commonly use:

```bash
sudo dpkg-reconfigure wireshark-common
sudo usermod -aG wireshark "$USER"
```

Log out and back in after changing group membership. Grant packet-capture privileges only as required by your lab; do not run arbitrary applications as root unnecessarily.

Official downloads and guide: <https://www.wireshark.org/download.html> and <https://www.wireshark.org/docs/wsug_html/>

### Step 2: Identify your interface

```bash
ip -br addr
ip route
```

Common interface names include `eth0` (Ethernet/virtual Ethernet) and `wlan0` (wireless). Use the interface that carries the traffic you want to observe. In the described Kali VirtualBox setup, `eth0` may be the active interface, but verify it on your machine.

List interfaces available to TShark:

```bash
tshark -D
```

### Step 3: Start a live capture

1. Launch Wireshark from the Kali Applications menu or run `wireshark`.
2. On the welcome page, double-click the active interface (for example, `eth0`).
3. Generate a small amount of known, ordinary lab traffic while capturing.
4. Stop the capture using the red square button.
5. Save it using **File → Save As**, choosing a descriptive name such as `ethical_hacking_lab.pcapng`.

For a controlled practice capture, open a second terminal and run:

```bash
nslookup example.com
ping -c 4 10.0.2.2
```

The second command uses the VirtualBox NAT gateway commonly seen in this setup; replace it with your actual gateway from `ip route` if different. If ICMP is filtered or the gateway does not respond, that does not by itself prove the network is down. DNS packets may appear if the DNS query uses the captured interface.

### Step 4: Read the three main panes

| Pane / field | Meaning and interpretation |
|---|---|
| **Packet List** | One row per packet. The usual columns include No., Time, Source, Destination, Protocol, Length, and Info. |
| **Packet Details** | Expandable protocol layers, such as Ethernet, IPv4/IPv6, TCP/UDP, DNS, HTTP, or TLS. |
| **Packet Bytes** | Raw packet bytes in hexadecimal and, where printable, ASCII. Useful for low-level inspection. |
| **Source / Destination** | Address endpoints for the currently selected protocol layer. They are not necessarily human identities. |
| **Protocol** | Protocol Wireshark has identified, for example DNS, ARP, TCP, UDP, ICMP, HTTP, or TLS. |
| **Info** | A concise summary, such as a DNS query, TCP SYN, TCP reset, or ICMP echo request. |

Double-click a packet or expand its protocol tree to inspect details. The exact rows and fields depend on the traffic captured.

### Step 5: Display filters to memorize

Enter a display filter in the filter bar and press Enter. A valid filter is normally shown in green; an invalid one is shown in red.

| Filter | What it displays |
|---|---|
| `dns` | DNS packets |
| `arp` | Address Resolution Protocol packets |
| `icmp` | ICMP traffic, such as ping requests/replies |
| `tcp` | TCP packets |
| `udp` | UDP packets |
| `http` | Decoded HTTP traffic where present |
| `tls` | TLS traffic, commonly used for HTTPS |
| `ip.addr == 10.0.2.15` | IPv4 packets where this address is source or destination |
| `ip.src == 10.0.2.15` | IPv4 packets whose source address matches |
| `ip.dst == 10.0.2.2` | IPv4 packets whose destination address matches |
| `tcp.port == 443` | TCP packets with source or destination port 443 |
| `dns.qry.name == "example.com"` | DNS queries for that name |
| `tcp.flags.syn == 1` | TCP packets with SYN flag set; can include connection attempts |
| `tcp.analysis.retransmission` | Packets Wireshark labels as TCP retransmissions |
| `tcp.flags.reset == 1` | TCP packets with the reset flag set |

### Capture filters vs display filters

- **Capture filter:** limits which packets are saved during capture; uses capture-filter syntax (BPF-style).
- **Display filter:** hides or shows packets in the current capture after they have been collected; uses Wireshark display-filter syntax.

Examples:

```text
Capture filter: host 10.0.2.15
Capture filter: port 53
Display filter: ip.addr == 10.0.2.15
Display filter: dns
```

Do not mix the two syntaxes. Official filter guide: <https://www.wireshark.org/docs/wsug_html_chunked/ChWorkDisplayFilterSection.html>.

### Step 6: Run a simple command-line capture with TShark (optional)

TShark is Wireshark's command-line packet analyzer.

```bash
# List interfaces
tshark -D

# Capture 50 packets from the selected interface and save them
sudo tshark -i eth0 -c 50 -w lab_capture.pcapng

# Read a saved capture and display only DNS packets
tshark -r lab_capture.pcapng -Y dns

# Read a capture and display packets involving a particular IP
tshark -r lab_capture.pcapng -Y 'ip.addr == 10.0.2.15'
```

Change `eth0` and the IP address to match your actual system. Use `-c` for a bounded capture in the lab; a long-running capture can create large files and collect sensitive information.

### Step 7: Interpret common observations

| Observation | Reasonable inference | What it does **not** prove by itself |
|---|---|---|
| DNS query followed by a DNS response | The client asked to resolve a name and a response was observed. | That the resulting website is safe or unsafe. |
| ICMP echo request and reply | The two endpoints exchanged ping traffic successfully. | That all ports/services on the endpoint are reachable. |
| TCP SYN → SYN/ACK → ACK | A TCP three-way handshake likely completed. | That the application itself worked correctly. |
| TCP RST | A connection was reset or refused at the TCP layer. | That the target is malicious. |
| Repeated retransmissions | Possible packet loss, congestion, capture loss, or a connectivity issue. | A single definitive cause without more context. |
| Many connection attempts to different ports in a short period | Could be consistent with port scanning or service discovery. | Malicious intent without source, timing, and context. |
| TLS traffic | Encrypted transport is being used. | The complete HTTP request/response content; HTTPS is normally not readable as plain text without appropriate decryption material. |

### Step 8: Practical write-up

**Aim:** To capture and analyze network packets using Wireshark.

**Procedure:** Select the active interface, start capture, generate known traffic, stop and save the capture, apply relevant display filters, inspect packet details, and document the observed endpoints/protocols.

**Result:** State the actual protocols and endpoint details observed in your capture. Do not copy example addresses or claim a protocol was observed if it did not appear.

### Wireshark viva questions

1. **What is Wireshark?** A packet capture and network protocol analysis tool.
2. **Is Wireshark active or passive?** Packet capture/analysis is generally passive; the traffic being analyzed may have been generated by other tools.
3. **What is a PCAP/PCAPNG file?** A file format for storing captured packet data; PCAPNG is the newer, more extensible capture format.
4. **What is a display filter?** An expression that controls which captured packets are shown.
5. **Difference between capture and display filters?** Capture filters select packets before/during collection; display filters select packets shown after capture.
6. **What does `dns` show?** Packets Wireshark decodes as DNS.
7. **Why can HTTPS content not normally be read?** TLS encrypts application data in transit; packet metadata and handshake information may still be visible.
8. **Does Wireshark block malicious traffic?** No. It observes and analyzes packets; prevention requires another control such as a firewall or IPS.
9. **What are the three main panes?** Packet List, Packet Details, and Packet Bytes.
10. **Why save a capture?** So it can be reviewed, filtered, shared with permission, and attached as evidence to the lab report.
11. **What is TShark?** Wireshark's command-line counterpart for capturing and analyzing packets.
12. **What does a TCP three-way handshake contain?** SYN, SYN-ACK, and ACK.
13. **What does a retransmission indicate?** A packet may have needed retransmitting; causes include loss, delay, congestion, capture loss, or duplicate analysis, so investigate surrounding packets.
14. **Why can Wireshark show only a few packets?** The selected interface may not carry the traffic, no traffic may have been generated, permissions may be insufficient, or the capture filter may be too restrictive.
15. **Can capture filters be changed after capture?** No; a capture filter determines what is recorded. Display filters can be applied/changed on the recorded capture.

---

# 📒 EXPERIMENT 5: Robtex – DNS, IP and Network Intelligence

---

## 5a. What is Robtex?

**Robtex** is a browser-based network-intelligence and OSINT service. It gathers and indexes public information about domains, IP addresses, hostnames, reverse DNS, WHOIS, Autonomous Systems (ASNs), and IP prefixes/BGP routes. It helps correlate infrastructure relationships; it is not primarily a vulnerability scanner.

Official website: <https://robtex.com/>

### Does Robtex need installation?

No. Open the website in a browser. It is useful on Windows or Kali and does not require a terminal command for the standard lookup workflow. Available data, tabs, and query limits can vary.

### Step 1: Search for a domain

1. Open <https://robtex.com/>.
2. Enter a domain assigned by your instructor, or use `example.com` to learn the interface.
3. Open the domain lookup results.
4. Review available DNS-related records, associated addresses, name servers, mail servers, and related hostnames.
5. Record the query and the actual results shown on screen.

### Step 2: Search for an IP address

Choose an IP address shown in the domain results and look it up in Robtex. Examine the details available, such as:

- Reverse DNS names (if records exist).
- Network owner / organization information.
- ASN (Autonomous System Number).
- Announced IP prefix / BGP route details.
- Related DNS or infrastructure records.
- Registration or reputation information where available.

### Step 3: Understand common records

| Field / record | Meaning | What it helps you investigate |
|---|---|---|
| **A** | Maps a hostname to an IPv4 address | IPv4 endpoint for a hostname |
| **AAAA** | Maps a hostname to an IPv6 address | IPv6 endpoint for a hostname |
| **MX** | Mail-exchange records | Mail-routing infrastructure |
| **NS** | Name-server records | DNS servers for a domain |
| **TXT** | Text stored in DNS | May contain SPF, verification, or other policy records |
| **CNAME** | Alias from one DNS name to another | DNS naming relationships |
| **PTR / reverse DNS** | Maps an IP to a hostname if configured | Possible host naming and infrastructure clues |
| **ASN** | Identifies an autonomous routing system | Network/operator relationship |
| **BGP prefix** | A block of IP addresses advertised through routing | Broader network range associated with an announcement |

### Step 4: How to write an inference

Use this structure:

> “The Robtex lookup for **[domain/IP searched]** displayed **[actual record or relationship]**. This suggests **[limited, evidence-based inference]**. The finding should be cross-checked with a DNS query or another reliable source because records can be incomplete, shared, cached, or outdated.”

**Example (illustrative only):** If several hostnames resolve to the same IP, you may infer that they share an address or hosting endpoint. You cannot conclude they belong to the same organization; they could be hosted behind a shared provider, proxy, or CDN.

### Step 5: Robtex vs similar tools

| Tool | Difference |
|---|---|
| Robtex | Correlates DNS, IP, reverse DNS, ASN, routing, and related public intelligence |
| WHOIS | Focuses on domain registration or IP allocation records |
| `nslookup` | Queries DNS through a resolver at the time of the query |
| Shodan | Searches indexed information about Internet-exposed devices and services |
| Nessus | Actively assesses target systems for potential vulnerabilities |

### Robtex practical write-up

**Aim:** To perform public network intelligence research on a domain or IP address using Robtex.

**Procedure:** Search the authorized/assigned domain, inspect DNS-related information, follow an associated IP lookup, identify any reverse DNS/ASN/prefix details available, and document evidence and limitations.

**Result:** Report only the records actually displayed and explain what they suggest about DNS and network relationships.

### Robtex viva questions

1. **What is Robtex?** A web-based OSINT/network-intelligence lookup service for domains, IPs, DNS, and routing information.
2. **Does Robtex require installation?** No, standard lookups use its website.
3. **What is reverse DNS?** A lookup from an IP address to a hostname, commonly using PTR records.
4. **What is an ASN?** An identifier for an autonomous system participating in Internet routing.
5. **What is a BGP prefix?** An advertised range of IP addresses in routing.
6. **Is Robtex a vulnerability scanner?** No, it primarily provides public network intelligence.
7. **Why cross-check the results?** Public indexes can be stale, incomplete, shared, or different from current DNS responses.
8. **What is the difference between Robtex and `nslookup`?** Robtex correlates indexed intelligence from multiple sources; `nslookup` queries DNS records through a resolver.
9. **What does an IP netblock/prefix tell you?** It indicates a range of IP addresses associated with a network announcement or allocation; it does not prove every address in that range belongs to the same service.
10. **Why might two domains map to one IP?** Shared hosting, reverse proxies, cloud platforms, or CDNs often host many names on a shared address.
11. **What is one limitation of Robtex?** The data may be cached, incomplete, or stale; verify key DNS results against an authoritative/current source when authorized.

---

# 📕 EXPERIMENT 6: Nessus – Vulnerability Assessment

---

## 6a. What is Nessus?

**Nessus**, developed by Tenable, is a vulnerability assessment scanner. It uses plugins to check systems and services for potential vulnerabilities, outdated software, missing patches, insecure settings, and other weaknesses. Findings can include severity, evidence, remediation advice, and CVE references when applicable.

Nessus mainly identifies and reports potential weaknesses. A result is not automatically proof that a system has been compromised, and findings should be validated.

Official download/documentation: <https://www.tenable.com/downloads/nessus> and <https://docs.tenable.com/nessus/Content/GettingStarted.htm>

### Step 1: Install Nessus on Windows (your assigned platform)

1. Download the Windows installer from Tenable's official downloads page.
2. Run the installer and follow the prompts.
3. Open a browser and go to **https://localhost:8834** on the same computer.
4. Complete the welcome/setup wizard.
5. Choose the edition you are licensed or authorized to use. For a student lab this may be Nessus Essentials, subject to current eligibility and licensing conditions.
6. Follow the official activation instructions, create the administrator account, and allow Nessus to download and initialize its plugin feed.

The first plugin initialization can take a significant amount of time. Do this early rather than immediately before the exam.

If the browser shows a certificate warning for the local service, verify that you are connecting to `https://localhost:8834` on your own machine. Do not ignore certificate warnings for unrelated remote websites.

### Step 2: Verify the Nessus service on Windows

If the web page does not load:

1. Press `Win + R`, type `services.msc`, and press Enter.
2. Look for the Tenable Nessus service and confirm it is running. Start or restart it if appropriate for your setup.
3. Reopen `https://localhost:8834`.

PowerShell may also show relevant services:

```powershell
Get-Service *Nessus*
```

### Optional: Install on Kali Linux

Only use this if your practical requires Linux installation. Download the correct package for your distribution and architecture from Tenable; do not guess a package filename.

For a downloaded Debian/Kali `.deb` package, the general commands are:

```bash
cd ~/Downloads
sudo dpkg -i Nessus-<version>-debian6_amd64.deb
sudo systemctl start nessusd
sudo systemctl status nessusd
```

Replace the placeholder filename with the exact downloaded filename. Once the service is running, browse to `https://localhost:8834` on that machine and finish the browser setup. Tenable installation guide: <https://docs.tenable.com/nessus/Content/InstallNessusLinux.htm>.

### Step 3: Create a Basic Network Scan

Use an IP address or hostname belonging to a system you own or have explicit permission to test. For a student exercise, use the instructor-assigned target or an intentionally vulnerable lab VM that is reachable on an isolated lab network.

1. Sign in to Nessus.
2. Select **New Scan**.
3. Choose **Basic Network Scan**.
4. Enter a descriptive name, such as `Lab Vulnerability Assessment`.
5. In **Targets**, enter the verified target IP or hostname.
6. Save the scan, review the configuration, and launch it.
7. Wait for completion; the duration depends on target reachability, enabled checks, and plugin initialization.

A Basic Network Scan is intended for general vulnerability assessment. Do not assume the target IP from a sample guide applies to your network.

### Step 4: Understand scan findings

| Severity | General meaning | Action |
|---|---|---|
| **Critical** | Potentially extremely serious issue | Validate promptly and prioritize remediation |
| **High** | Significant potential risk | Prioritize investigation and fixing |
| **Medium** | Moderate potential risk | Assess and remediate according to context |
| **Low** | Lower-severity issue | Review and address where appropriate |
| **Info** | Observation or inventory detail; not always a vulnerability | Use as context for the assessment |

The exact score/severity depends on Nessus settings and vulnerability scoring data. Severity alone is not enough; consider evidence, asset importance, exposure, exploitability, and compensating controls.

### Step 5: Open a finding and interpret its details

For each important finding, record:

- Finding/plugin title.
- Severity and score, if shown.
- Affected host and port/service.
- Description and evidence/output.
- CVE identifier, if one is listed.
- Suggested solution or remediation.
- Whether the result has been validated or still needs confirmation.

**CVE** stands for Common Vulnerabilities and Exposures. A CVE identifier refers to a publicly catalogued vulnerability; not every finding has a CVE, and the presence of a CVE reference does not by itself prove that the scanned system is affected.

**Plugin** is a Nessus test used to detect a vulnerability or collect security-related information. Keep plugin feed updates in mind when interpreting the results.

### Step 6: Save evidence / export results

Open the completed scan and inspect its available **Export** or **Report** options. Export formats depend on the installed Nessus edition and current licensing. Tenable's current Nessus documentation states that standard Nessus Essentials does not offer the same scan reports/exports as paid editions; available options can vary by product/version. If your edition does not provide a report export, record the scan summary and finding details in your lab notes and take permitted screenshots.

Official references: [Basic Network Scan template](https://docs.tenable.com/nessus/Content/ScanAndPolicyTemplates.htm) and [scan reports/exports](https://docs.tenable.com/nessus/Content/ScanReportFormats.htm).

### Common problems and fixes

| Problem | Checks / response |
|---|---|
| `localhost:8834` does not open | Verify the Nessus service is running; wait for startup; check the exact local URL. |
| Plugin initialization is still running | Allow it to finish and check service/network connectivity; plugin updates can take time. |
| Target shows as unreachable | Verify the IP address, VM network mode, routing, and whether the target is powered on. |
| Scan completes with no findings | This does not prove the system is secure. Check reachability, scope, credentials (if relevant), enabled plugins, and scan coverage. |
| A finding looks incorrect | Read the evidence and affected version, then validate safely; scanners can produce false positives. |
| Export/report option is absent | Check your edition/licence. Record the findings manually if export is not available. |

### Nessus practical write-up

**Aim:** To perform vulnerability assessment on an authorized host using Nessus.

**Procedure:** Open Nessus, complete plugin initialization, create a Basic Network Scan, specify the authorized target, launch the scan, review severity and plugin details, record evidence and remediation, and save available results.

**Result:** State the number/types of findings actually observed and their severities. If the scan could not finish, describe the limitation instead of inventing results.

### Nessus viva questions

1. **What is Nessus?** An automated vulnerability assessment scanner.
2. **What is a Nessus plugin?** A test used to detect a weakness or gather security information.
3. **What is CVE?** Common Vulnerabilities and Exposures, a standardized identifier for publicly known vulnerabilities.
4. **What does Basic Network Scan do?** Performs a general vulnerability assessment against the specified host(s).
5. **What is a false positive?** A reported issue that validation shows is not actually present or applicable.
6. **Does a clean scan guarantee security?** No. Coverage, credentials, plugin freshness, configuration, and scan limitations matter.
7. **Nessus vs Nmap?** Nmap discovers hosts, ports, and services; Nessus focuses on potential vulnerabilities and their remediation.
8. **What is remediation?** The corrective action that reduces or removes the weakness.
9. **Why does Nessus need a plugin feed?** Plugins provide the checks and vulnerability knowledge used by the scanner.
10. **Is Nessus passive?** No, vulnerability assessment scans generally interact with target systems and require authorization.
11. **What is CVSS?** Common Vulnerability Scoring System, a standardized way of expressing vulnerability severity; a score is not the same as complete business risk.
12. **What is a credentialed scan?** A scan using authorized credentials to inspect local patch/configuration state more deeply than an unauthenticated scan may allow.
13. **Why is plugin initialization important?** The scanner uses its plugins to conduct checks; an incomplete or stale feed can reduce coverage.
14. **What should you write in the practical result?** The actual scan target, number/severity of findings if available, one finding's evidence and remediation, and any coverage limitation.
15. **If Nessus finds nothing, is the target secure?** No. Empty results may reflect limited visibility, missing credentials, plugin/scan settings, target reachability or real lack of detected issues.

---

# 📔 EXPERIMENT 7: Armitage – Metasploit GUI and Lab Exploitation Management

---

## 7a. What is Armitage?

**Armitage** is a graphical front end for the Metasploit Framework. It visualizes hosts, helps organize assessment data, provides access to Metasploit modules, and can manage sessions. It communicates with Metasploit through its RPC service; Metasploit provides the modules and execution engine.

- **Kali Linux:** the assessment machine that runs Armitage and Metasploit.
- **Metasploitable / another deliberately vulnerable VM:** the target for authorized lab exercises.
- **VirtualBox isolated network:** connects the lab machines without exposing the vulnerable target to the public Internet.

Armitage is not an OSINT search engine or a vulnerability scanner in the same sense as Nessus. It is a GUI for managing Metasploit workflows, including controlled testing of applicable exploits in an authorized lab.

Lab manual workflow: verify installation, start the database/services, connect Armitage to Metasploit RPC, create a workspace, discover hosts, inspect services, use **Find Attacks**, and document the authorized lab result.

### Step 1: Check whether it is installed

```bash
command -v armitage
armitage --help
msfconsole -v
sudo msfdb status
```

If `command -v armitage` returns a path and `armitage --help` prints usage information, the executable is installed. Do not reinstall if these checks succeed.

### Step 2: Install only if it is missing

On Kali, check repository availability before installing:

```bash
sudo apt update
apt-cache policy armitage
```

If the package is available in your configured Kali repositories:

```bash
sudo apt install armitage metasploit-framework postgresql
```

The exact availability and dependencies may vary with the current Kali release and repository configuration. If `apt` says there is no installation candidate, do not download random installers; check the lab manual and Kali repository configuration first.

### Step 3: Start and verify the database

```bash
sudo systemctl start postgresql
sudo systemctl status postgresql
sudo msfdb status
```

If the Metasploit database has **never** been initialized, follow the lab's first-run procedure, which may include:

```bash
sudo msfdb init
```

Use initialization only for a new setup; do not reset or reinitialize an existing database just to troubleshoot without understanding the consequences.

### Step 4: Launch Armitage and connect

```bash
sudo armitage
```

When the connection dialog appears, the local RPC defaults in the lab manual are usually:

- Host: `127.0.0.1`
- Port: `55553`

Use the default lab credentials/settings presented by your own version and allow Armitage to start its local RPC service if prompted. Wait for the first connection, as Metasploit may need time to load modules. Do not expose the RPC service to an untrusted network.

### Step 5: Understand the interface

| Component | Purpose |
|---|---|
| **Targets panel** | Visual icons for hosts discovered or imported into the current workspace |
| **Modules panel** | Search/browse auxiliary, exploit, payload, and post modules |
| **Console/session tabs** | Display commands, module output, and active sessions |
| **Workspaces** | Keep the hosts, services, and sessions for separate lab exercises organized |
| **Hosts menu** | Add/import hosts and initiate supported host discovery/scan workflows |
| **Attacks menu** | Contains features such as **Find Attacks**; features vary with version |

### Step 6: Create a lab workspace and identify the host

1. Choose **Workspaces → Manage** (wording may vary) and create a workspace such as `lab-week9-armitage`.
2. Ensure your instructor-assigned target is powered on and reachable.
3. If your target IP is known, add only that authorized lab host. Alternatively, use **Hosts → Nmap Scan** for a specifically authorized lab subnet.
4. Open the host's details/services and record discovered ports, services, and version information where available.

**VirtualBox networking warning:** Two VMs both configured for ordinary NAT may not automatically be able to communicate directly. Use the network mode specified by your instructor—commonly the same host-only or internal network on both lab VMs—and verify their actual IP addresses. Avoid bridged networking for deliberately vulnerable images because it may expose them to a real network.

### Step 7: Use Find Attacks

Select the authorized lab host, then choose **Attacks → Find Attacks**. Armitage compares recorded host/service information with its available module database. The first run can take longer while the module list is indexed.

Interpretation: the suggested module list contains candidates that may fit the observed services. It is not proof that an exploit works, and service/version detection may be incomplete or wrong.

### Step 8: Controlled module testing and session indicators

For an instructor-authorized, intentionally vulnerable target, follow the instructor's specified module and options. Before launching any test, verify the target host and port, the module's target/version assumptions, and that the listener settings are appropriate for the isolated lab. Active exploitation can crash services or modify the target, so do not experiment against shared or real systems.

A successfully created session may be shown as a changed host icon and a session tab in Armitage. If no session is created, that does not automatically mean there are no vulnerabilities: the module may not apply, the target/version may differ, connectivity or listener configuration may be wrong, or the target may be patched.

Avoid **Attacks → Hail Mary** except where an instructor explicitly asks for it in an isolated disposable lab. It attempts multiple applicable exploits and is noisy and less selective.

### Step 9: Basic troubleshooting

| Error / symptom | Checks |
|---|---|
| Armitage cannot connect to `127.0.0.1:55553` | Check `sudo msfdb status`, PostgreSQL status, and the RPC startup prompt; allow the service time to start and retry. |
| PostgreSQL is stopped | Run `sudo systemctl start postgresql`, then check `sudo systemctl status postgresql`. |
| `armitage: command not found` | Check `apt-cache policy armitage`; verify whether the package exists in the configured Kali repositories. |
| No target discovered | Confirm target is running, the IP/subnet is correct, and both VMs use the intended lab network. A target may block ping probes. |
| `Find Attacks` is slow | The initial module indexing can be slow; wait and check that Metasploit has initialized correctly. |
| No session appears | Check service/version applicability, permitted target settings, reachability, and module output; do not repeatedly run arbitrary exploits. |

### Armitage practical write-up

**Aim:** To explore Armitage as a GUI for organizing authorized targets and managing Metasploit assessment workflows in an isolated lab.

**Procedure:** Verify the Armitage/Metasploit installation, start and check PostgreSQL/Metasploit database services, launch Armitage and connect to local RPC, create a workspace, identify the instructor-assigned lab host, review services, and use **Find Attacks** to inspect candidate modules. Perform only the controlled action specified by the instructor.

**Result:** Document the host/service information, the modules suggested, and the actual lab outcome. State clearly whether a session was created; do not equate a suggested exploit with a successful exploit.

### Armitage viva questions

1. **What is Armitage?** A graphical front end for managing Metasploit targets, modules, and sessions.
2. **What is Metasploit?** A penetration-testing framework that contains auxiliary, exploit, payload and post modules.
3. **What is RPC?** Remote Procedure Call, used by the Armitage client to communicate with the Metasploit service.
4. **Why create a workspace?** To keep data from individual exercises organized and separate.
5. **What does Find Attacks do?** Matches gathered host/service information against available exploit modules to suggest candidates.
6. **What is Meterpreter?** A Metasploit payload/session interface used for authorized post-exploitation tasks.
7. **Does a suggested attack mean the target is vulnerable?** No. It must be checked against evidence and the real target configuration.
8. **Why use Metasploitable?** It is an intentionally vulnerable lab target for learning in an isolated, authorized environment.
9. **Why isolate the target VM?** To reduce risk of exposing vulnerable services or affecting real systems.
10. **What is the difference between Nessus and Armitage?** Nessus identifies/reports potential weaknesses; Armitage manages Metasploit workflows that may be used to validate specific weaknesses in a lab.
11. **What is a payload?** Code delivered/started by an exploit or module to provide a defined function in a controlled test; payload selection must fit the target and the authorized exercise.
12. **What is a Meterpreter session?** An interactive Metasploit session with specialized post-exploitation commands; it should be used only on an explicitly authorized lab host.
13. **Does the host icon changing color prove the entire host is compromised?** It indicates Armitage's recorded host/session state; interpret it alongside the console output and actual session information.
14. **Why can Find Attacks show a module that fails?** The module may rely on an inaccurate service/version fingerprint, target conditions may differ, or network/payload settings may be unsuitable.
15. **Why not use Hail Mary automatically?** It attempts many candidate modules and is noisy, less selective, and can disrupt services; use it only if explicitly required in a disposable isolated lab.

---

# 🧠 Scenario Selector – Which Tool Should I Choose?

| Scenario in the practical question | Best-fit tool(s) | What to show or report |
|---|---|---|
| Need process creation and endpoint connection logs on Windows | **Sysmon** | Event Viewer path, Event IDs, event fields and interpretation |
| Need public emails, hostnames, or subdomains | **theHarvester** | Source used, findings, saved output, and passive-recon explanation |
| Need live IP and MAC addresses on the same LAN | **Netdiscover** | Interface, authorized subnet, IP/MAC/vendor columns |
| Need public documents or indexed admin/login pages | **Google Dorking** | Search operators and relevant public results, without accessing protected data |
| Need author names, software, usernames, internal paths from office documents | **FOCA** | Document discovery, downloaded files, extracted metadata and leakage risk |
| Need registrar and domain registration dates | **WHOIS** | Registrar, creation/expiry, name servers, redacted contacts if present |
| Need a specific DNS record | **nslookup** | Query and record type: A/AAAA/MX/NS/SOA/TXT/CNAME/PTR |
| Need a broader list of subdomains from public sources | **Sublist3r** | Command, data sources, deduplicated findings and output file |
| Need a visual relationship graph of domains, people, emails, and IPs | **Maltego (Windows desktop)** | Graph, entities, transforms and relationship explanation |
| Need indexed Internet-facing device/service information | **Shodan (browser)** | Query/filter, open service/port/banner data and record freshness caveat |
| Need web-server misconfiguration/known-file checks | **Nikto** | Target, scan output, interesting paths/headers, recommendations |
| Need host discovery, open ports, service versions, possible OS fingerprint | **Nmap** | Command, port state/service table and scan limitations |
| Need decoded packets and network troubleshooting evidence | **Wireshark** | Capture interface, filters, relevant packets and interpretation |
| Need public DNS/IP/ASN/reverse-DNS relationships | **Robtex** | Lookup records, IP/ASN/prefix and evidence limitations |
| Need vulnerability plugin findings, severity and remediation | **Nessus** | Scan type, target, severity, plugin evidence, CVE (if listed), remediation |
| Need GUI access to Metasploit modules and lab sessions | **Armitage** | Workspace, target/service details, Find Attacks output, actual session state |

---

# 🎓 Consolidated Viva Bank – 16 Tools

## A. Reconnaissance and OSINT

1. **What is reconnaissance?** The information-gathering phase used to understand a target's public-facing or internal environment before further assessment.
2. **Active vs passive reconnaissance?** Active methods interact directly with the target and may be logged; passive methods use public records, indexed data, or third-party sources. Some tools support both modes.
3. **Which tool gathers emails from public sources?** theHarvester. In this lab it is run from the Kali terminal.
4. **Which tool discovers local IP/MAC mappings using ARP?** Netdiscover.
5. **What is Google Dorking?** Using advanced search operators to find indexed public pages and files.
6. **What is metadata?** Data describing a file, such as its author, software, timestamps or internal path.
7. **What is the main purpose of FOCA?** Discover public documents and extract metadata that may reveal organization details.
8. **What does WHOIS provide?** Domain/IP registration or allocation details, subject to registrar and registry privacy/redaction.
9. **What is DNS?** The Domain Name System, which maps names to records such as IP addresses and mail servers.
10. **What is the purpose of an MX record?** It identifies mail servers used for a domain's email delivery.
11. **What does Sublist3r enumerate?** Subdomains associated with a domain, primarily through public sources unless active options are used.
12. **What is a Maltego transform?** An automated query/action used from the Windows desktop application to obtain related entities and add them to a graph.
13. **What is Shodan?** A browser-based search engine for indexed Internet-connected devices and exposed services in this practical.
14. **Does Shodan data always represent the current state?** No. Check its last-seen/update information and verify when authorized.
15. **Does Robtex scan a host for vulnerabilities?** No. It primarily collects and correlates public network intelligence.

## B. Scanning, Monitoring, and Analysis

16. **What is the difference between Nmap and Nikto?** Nmap focuses on host/port/service discovery; Nikto checks web servers for known files, outdated software, and common configuration issues.
17. **What does `-sV` do in Nmap?** Attempts service/version detection on discovered ports.
18. **What does `-sn` do?** Performs host discovery without the normal port scan.
19. **What does an open port mean?** A service appears to be accepting connections on that port; the service and exposure still need contextual assessment.
20. **What is a filtered port?** Nmap cannot determine the port state reliably because filtering or packet behavior prevents a clear answer.
21. **What is Nessus used for?** Automated vulnerability assessment and reporting potential issues with evidence and fixes.
22. **What is a false positive?** A finding reported by a tool that is not confirmed to apply to the target.
23. **What is Sysmon?** A Windows host-monitoring service/driver that records detailed activity into the Windows Event Log.
24. **Which Sysmon Event IDs are especially useful for process creation, network connection, and DNS queries?** Event IDs 1, 3, and 22 respectively, if those events are enabled by the configuration.
25. **What does Wireshark do?** Captures and analyzes network packets and decoded protocol fields.
26. **Can Wireshark show every device's traffic on a switched LAN?** Not usually; it only sees traffic visible to the selected interface unless capture is arranged appropriately.
27. **What is the difference between Wireshark and Nessus?** Wireshark analyzes captured network traffic; Nessus actively assesses systems for potential vulnerabilities.
28. **What is Armitage?** A GUI front end to Metasploit for target visualization, module use, and session management.
29. **What does a Nessus CVE field mean?** The finding references a known vulnerability identifier; you must still check applicability and evidence.
30. **Why should vulnerable VMs be isolated?** To prevent accidental exposure or impact on networks and devices outside the exercise.

## C. Practical answering pattern

For a scenario-based 6- or 12-mark answer, use the sequence below and adapt it to the question:

1. **Aim / objective:** Say what the tool is being used to discover, monitor, or validate.
2. **Requirements:** State the OS/application, permissions, target/domain/subnet, and lab network needed.
3. **Procedure:** List setup steps and the main command or GUI menu sequence.
4. **Evidence/output:** Include the meaningful screen/table/log fields actually produced.
5. **Interpretation:** Explain what the evidence means and what it cannot establish.
6. **Security relevance:** State the risk or why the observation matters.
7. **Mitigation / recommendation:** Provide a suitable corrective action if a weakness was found.
8. **Result:** One short sentence summarizing what you actually completed.

## D. Common practical mistakes to avoid

- Do not substitute sample IPs or sample output counts for your actual results.
- Do not scan a random range. Verify the lab subnet and target address first.
- Do not call every open port a vulnerability or every finding a confirmed exploit.
- Do not claim a result was found if the command failed, the tool was still initializing, or no output was returned.
- Do not confuse a capture filter with a Wireshark display filter.
- Do not assume every Robtex/Shodan record is current; note that indexed data can lag.
- Do not forget permissions, plugin feeds, required database services, or service reachability.
- Save evidence where permitted and make sure screenshots show the relevant tool, target, and result.
- In each answer, explain **why the finding matters**, not merely what the tool displayed.

---

# 📋 Quick Reference – All 16 Tools Summary

| # | Tool | Platform for this practical | Primary purpose | Basic action / command | Active/passive note |
|---:|---|---|---|---|---|
| 1 | **Sysmon** | Windows | Endpoint/process/network/file telemetry in Windows Event Log | `Sysmon64.exe -accepteula -i sysmonconfig-export.xml` | Local monitoring/logging; does not itself block threats |
| 2 | **theHarvester** | Kali terminal | Public emails, hostnames, and subdomains | `theHarvester -d example.com -l 100 -b <supported-backend>` | Mainly passive public-source OSINT |
| 3 | **Netdiscover** | Kali terminal | Discover local IPv4/MAC/vendor information via ARP | `sudo netdiscover -i eth0 -r <authorized-subnet>` | Active or passive mode |
| 4 | **Google Dorking** | Browser | Search indexed public pages/files with operators | `site:example.com filetype:pdf` | Search-based/passive reconnaissance |
| 5 | **FOCA** | Windows desktop GUI | Find public documents and extract metadata | Project → Search → Download → Extract Metadata | Primarily public-source/document analysis |
| 6 | **WHOIS** | Kali terminal | Domain registration or IP allocation information | `whois example.com` | Registration-record lookup |
| 7 | **nslookup** | Kali terminal | Resolve names and query DNS records | `nslookup -type=mx example.com` | DNS query to resolver |
| 8 | **Sublist3r** | Kali terminal | Enumerate publicly discoverable subdomains | `sublist3r -d example.com -o subdomains.txt` | Mainly passive; brute force/port checks are active |
| 9 | **Maltego** | Windows desktop GUI | Visualize links among domains, IPs, email, people and other entities | New Graph → Domain entity → Run Transforms | Most OSINT transforms query sources; check each transform |
| 10 | **Shodan** | Browser | Search indexed Internet-connected device/service information | `hostname:your-authorized-domain.example` | Querying indexed data; may be stale |
| 11 | **Nikto** | Kali terminal | Check web server for known files, software clues and configuration issues | `nikto -h http://<authorized-lab-web-server>` | Active and potentially noisy |
| 12 | **Nmap** | Kali terminal | Host discovery, ports, services, version and OS hints | `nmap -sV <authorized-lab-target>` | Active scanning |
| 13 | **Wireshark** | Kali GUI (TShark optional) | Capture and analyze network packets | Select interface → Capture → display filter such as `dns` | Observes visible traffic; does not block it |
| 14 | **Robtex** | Browser | Correlate public DNS/IP/reverse-DNS/ASN/prefix information | Search an assigned domain or IP at `robtex.com` | Public/indexed network intelligence |
| 15 | **Nessus** | Windows service + browser UI | Assess potential vulnerabilities, severity and remediation | `https://localhost:8834` → New Scan → Basic Network Scan | Active vulnerability scanning |
| 16 | **Armitage** | Kali GUI | Manage Metasploit targets, modules and sessions in an authorized lab | `sudo armitage` | Active when scanning/testing targets |

> **Platform memory trick:** Windows = FOCA, Nessus, Sysmon, Maltego. Kali = WHOIS, nslookup, Nmap, Armitage, theHarvester, Netdiscover, Sublist3r, Wireshark, plus Nikto retained from the original guide. Browser = Robtex, Shodan, Google Dorking.

### 30-second platform self-test
Before the exam, say these without looking: **Windows:** FOCA, Nessus, Sysmon, Maltego. **Kali:** WHOIS, nslookup, Nmap, Armitage, theHarvester, Netdiscover, Sublist3r, Wireshark, Nikto. **Browser:** Robtex, Shodan, Google Dorking. Remember: Nessus is opened *in a browser* but its scanner service is installed on Windows; Wireshark/Armitage are graphical tools used in Kali; Shodan and Robtex are browser-first in this practical.

---
---

# 🎯 Exam Strategy Tips

> [!IMPORTANT]
> **Must-Remember for Your Exam:**
> 1. **Read the question carefully** — note whether it says "authorized domain/network"
> 2. **Start with the basic command** — then add flags for more detail
> 3. **Always show and explain the output** — don't just run commands, interpret results
> 4. **Save your outputs** — use `-o` or `-f` flags when available
> 5. **Know Active vs. Passive** — Nmap, Nikto and Nessus actively probe targets; Armitage can scan/test targets; Netdiscover has active and passive modes. WHOIS, Robtex and Shodan lookups generally use public/indexed data. theHarvester/Sublist3r are mainly passive unless you enable active options. Wireshark usually observes traffic.

> [!WARNING]
> **Key Difference to Remember:**
> - **Passive** = Using public records/indexes (for example, WHOIS, Robtex, and querying Shodan data) without directly probing the target in that step.
> - **Active** = Sending packets or requests to the target (for example, Nmap, Nikto, Nessus, or Armitage scans); these actions may be logged. Netdiscover supports active and passive modes, and Wireshark usually observes traffic rather than probing a target.

> [!CAUTION]
> **Never run active scans (including Nmap, Nikto, Nessus, or Armitage testing) on unauthorized targets.** In your exam, scan only instructor-approved lab machines, VMs, networks, or explicitly authorized domains. Unauthorized scanning is **illegal**.

---

# 🧪 Final platform-specific launch checklist

Use this before the practical begins; it is intentionally short enough to revise quickly.

## Windows
- **Sysmon:** open Command Prompt/PowerShell as Administrator when installing or changing config; verify service state; read `Microsoft-Windows-Sysmon/Operational` in Event Viewer.
- **FOCA:** start the GUI; confirm its SQL Server connection; create a project; search for documents; extract metadata. If database connection fails, check SQL Server service/instance and authentication.
- **Nessus:** confirm Tenable Nessus service is running; open `https://localhost:8834`; let plugin initialization finish; create Basic Network Scan; use only the verified lab target.
- **Maltego:** open the Windows desktop app; sign in; create a graph; add a Domain entity; run an available transform; explain each relationship and the transform's source/limitations.

## Kali Linux terminal / GUI
- **WHOIS:** `whois example.com` — record registrar/creation/expiry/name servers when returned.
- **nslookup:** `nslookup example.com` then `nslookup -type=mx example.com` — identify A/AAAA or MX results.
- **Nmap:** `nmap -sn <authorized-subnet>` for host discovery or `nmap -sV <authorized-target>` for services; do not blindly default to `-A`.
- **Armitage:** `sudo msfdb status` → `sudo systemctl status postgresql` → `sudo armitage`; add only the authorized lab host.
- **theHarvester:** `theHarvester -h` first to check available backends, then run with the assigned domain and a supported backend.
- **Netdiscover:** check `ip -br addr` and choose the verified interface/subnet; remember ARP works on the local link.
- **Sublist3r:** `sublist3r -h` then `sublist3r -d <authorized-domain> -o subdomains.txt`.
- **Wireshark:** open GUI, choose active interface, generate ordinary lab traffic, stop/save, then apply `dns`, `arp`, `icmp`, or another correct display filter.
- **Nikto (retained from the original tool list):** `nikto -h http://<authorized-lab-web-server>`; interpret output as leads requiring validation.

## Browser
- **Robtex:** inspect a domain and a related IP; identify DNS/ASN/reverse-DNS details and mention stale/shared-hosting caveats.
- **Shodan:** search only an authorized hostname/organization; read ports, product/banner, last-seen/update details; explain it is indexed data, not guaranteed live state.
- **Google Dorking:** use `site:`, `filetype:`, `intitle:`, `inurl:` with the authorized domain; record the visible result and recommend removing unintended exposure.

# 🎤 Rapid-fire viva: one-line comparisons

- **FOCA vs Google Dorking:** Google finds indexed pages/files; FOCA analyzes documents and extracts metadata.
- **WHOIS vs nslookup:** registration/allocation records vs live DNS-record queries through a resolver.
- **theHarvester vs Sublist3r:** theHarvester can gather emails, hosts and subdomains; Sublist3r focuses on subdomain enumeration.
- **Maltego vs Robtex:** graph-based entity relationship analysis vs browser lookup/correlation of DNS, IP and routing information.
- **Shodan vs Nmap:** search an index of Internet-connected assets vs actively probe specified hosts/ports from your machine.
- **Nmap vs Nessus:** discover hosts/ports/services vs assess possible vulnerabilities and provide remediation guidance.
- **Nikto vs Nessus:** web-server-focused checks vs broader host/network vulnerability assessment.
- **Wireshark vs Netdiscover:** packet/protocol capture and analysis vs local ARP-based host discovery.
- **Sysmon vs Wireshark:** endpoint event telemetry from Windows vs network packets visible to a capture interface.
- **Nessus vs Armitage:** identify/report possible weaknesses vs orchestrate Metasploit modules/sessions for controlled validation in a lab.
- **Passive vs active:** public/indexed data collection vs direct target interaction; tools can have modes that change this classification.
- **Vulnerability vs risk:** a weakness is a technical issue; risk also depends on likelihood, impact, exposure, asset value and compensating controls.
- **False positive vs false negative:** incorrectly reported finding vs missed real issue.
- **Evidence vs conclusion:** output is evidence; the conclusion must explain what it supports and what remains unverified.

---

# 🔄 Exam Flow – What to Do Step by Step

```
1. Read the question → Identify the tool
2. Open terminal (Kali) or application (Windows)
3. Run the basic command with the target
4. Wait for output
5. Analyze and explain what each finding means
6. Save output to a file (for documentation)
7. Summarize: what you found, why it matters
```

---

*Use this guide as a practical checklist, not a guarantee of a mark. Tool interfaces and versions may differ; always describe the output you actually observe rather than memorizing sample IPs or counts.*


# 🔗 Official documentation and source links

Use these when installation steps or menus differ from your installed version. These links supplement the lab manual; your instructor's stated platform/workflow takes precedence for the practical.

- **Sysmon:** <https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon>
- **FOCA project and prerequisites:** <https://github.com/ElevenPaths/FOCA>
- **Maltego desktop and transforms:** <https://www.maltego.com/downloads/> and <https://docs.maltego.com/>
- **Nessus Windows installation:** <https://docs.tenable.com/nessus/Content/InstallNessusWindows.htm>
- **Nessus plugin/scan and reporting docs:** <https://docs.tenable.com/nessus/Content/GettingStarted.htm>
- **Kali Nmap tool page:** <https://www.kali.org/tools/nmap/>
- **Kali Netdiscover tool page:** <https://www.kali.org/tools/netdiscover/>
- **Kali Sublist3r tool page:** <https://www.kali.org/tools/sublist3r/>
- **Kali Nikto tool page:** <https://www.kali.org/tools/nikto/>
- **Wireshark User's Guide / display filters:** <https://www.wireshark.org/docs/wsug_html/> and <https://www.wireshark.org/docs/wsug_html_chunked/ChWorkDisplayFilterSection.html>
- **Shodan browser search:** <https://www.shodan.io/> and help: <https://help.shodan.io/>
- **Robtex:** <https://robtex.com/>

---
