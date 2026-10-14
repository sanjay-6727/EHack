# 🔐 Ethical Hacking Lab Exam – Complete Study Guide
## ICS1561 / UCS3568 | Total Marks: 50

> [!IMPORTANT]
> **All tools must be used ONLY on authorized targets/networks you have explicit permission to test.**
> Using these tools on unauthorized systems is illegal.

## Quick Navigation

- [Last-Minute Revision Sheet](#last-minute-revision-sheet)
- [Windows tools](#windows-tools) — FOCA, Nessus, Sysmon, Maltego
- [Kali Linux tools](#kali-linux-tools) — WHOIS, nslookup, Nmap, Armitage, theHarvester, Netdiscover, Sublist3r, Wireshark
- [Browser-based tools](#browser-based-tools) — Robtex, Shodan, Google Dorking
- [All-Tools Comparison](#-quick-reference--all-tools-summary)
- [Exam Strategy](#-exam-strategy-tips)

## Tool Interface References

These links lead to official product pages or documentation containing real interface images/screenshots. External images are used so the README stays lightweight.

| Tool | Visual reference | Official resource |
|---|---|---|
| FOCA | [FOCA project and interface references](https://github.com/ElevenPaths/FOCA) | [GitHub repository](https://github.com/ElevenPaths/FOCA) |
| Nessus | [Nessus dashboard screenshot/documentation](https://docs.tenable.com/nessus/Content/Dashboard.htm) | [Tenable Nessus docs](https://docs.tenable.com/nessus/) |
| Sysmon | [Sysmon screenshots and event examples](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) | [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) |
| Maltego | [Maltego Graph interface](https://www.maltego.com/graph/) | [Maltego Graph](https://www.maltego.com/graph/) |
| Armitage | [Armitage interface walkthrough](https://www.computerweekly.com/tutorial/Metasploit-guide-4-Armitage-cyber-attack-management-GUI) | [Armitage overview](https://www.usenix.org/system/files/login/issues/login_aug_2011.pdf) |
| Wireshark | [Wireshark interface and screenshots](https://www.wireshark.org/) | [Wireshark documentation](https://www.wireshark.org/docs/) |
| Shodan | [Shodan search interface](https://www.shodan.io/) | [Shodan](https://www.shodan.io/) |
| Robtex | [Robtex DNS lookup example](https://robtex.com/en/dns-lookup/com/robtex/dns) | [Robtex](https://www.robtex.com/) |

---

# Last-Minute Revision Sheet

## Memorize the purpose in one line

| Platform | Tool | Remember this |
|---|---|---|
| Windows | **FOCA** | Finds metadata in public documents. |
| Windows | **Nessus** | Scans for known vulnerabilities and misconfigurations. |
| Windows | **Sysmon** | Logs detailed Windows host activity; it does not block threats. |
| Windows | **Maltego** | Maps relationships between entities in a graph. |
| Kali | **WHOIS** | Domain/IP registration and allocation information. |
| Kali | **nslookup** | Queries DNS records such as A, MX, NS and TXT. |
| Kali | **Nmap** | Discovers hosts, ports, services and sometimes OS details. |
| Kali | **Armitage** | GUI for working with the Metasploit Framework. |
| Kali | **theHarvester** | Collects public OSINT such as emails and hostnames. |
| Kali | **Netdiscover** | Discovers local-network hosts using ARP. |
| Kali | **Sublist3r** | Enumerates subdomains from public sources. |
| Kali | **Wireshark** | Captures and analyzes network packets. |
| Browser | **Robtex** | Online DNS, IP and network relationship lookup. |
| Browser | **Shodan** | Searches its index of internet-connected devices and exposed services. |
| Browser | **Google Dorking** | Uses search operators to find indexed pages/files. |

## Active vs. passive — common exam distinction

- **Usually passive/public-source collection:** WHOIS, Google Dorking, Shodan queries, Robtex, FOCA (searching/downloading already-public documents), theHarvester and Sublist3r when using passive sources.
- **Active interaction/scanning:** Nmap, Nikto, and active Netdiscover.
- **Local host monitoring:** Sysmon (records events on the Windows machine).
- **Important nuance:** a tool's mode depends on the feature and data source used. For example, Sublist3r brute-force mode and some enumeration options can actively query hosts.

## Command flash cards

```bash
whois example.com
nslookup -type=mx example.com
nmap -sV 192.168.1.10
sudo msfdb init
armitage
theHarvester -d example.com -l 100 -b crtsh
sudo netdiscover -r 192.168.1.0/24
sublist3r -d example.com -o subdomains.txt
wireshark
```

Use only a lab, localhost, or a target/network for which you have explicit authorization. `example.com` and the private IP range above are illustrative; replace them only in an authorized lab.

---

## WINDOWS TOOLS

### 1. FOCA — Metadata Extraction from Documents

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
> - FOCA is a **Windows-only** GUI tool

---

---

### 2. Nessus — Vulnerability Assessment

**Real interface screenshot:** [Nessus dashboard in the official documentation](https://docs.tenable.com/nessus/Content/Dashboard.htm)

**Purpose:** Nessus is a vulnerability scanner that checks authorized systems for known vulnerabilities, missing patches, exposed services and configuration weaknesses using plugins/checks.

**Basic workflow**
1. Open the Nessus web interface and sign in.
2. Create a scan template (for example, Basic Network Scan).
3. Enter only an authorized lab target.
4. Configure credentials/options if required and launch the scan.
5. Review findings by severity, affected host, plugin and remediation.
6. Export a report if the lab requires evidence.

**Know for the exam**
- Nessus is a **vulnerability assessment** tool, not just a port scanner.
- Results commonly include severity, plugin details, affected assets, evidence and remediation guidance.
- A detected version or banner is not always proof that a system is exploitable; validate findings.
- Scans can be credentialed or non-credentialed. Credentialed scans can often inspect patch/configuration state more accurately.
- Do not scan public or third-party systems without permission.

**Typical exam answer:** “Nessus scans authorized hosts for known vulnerabilities and misconfigurations. It reports findings with severity and remediation guidance so administrators can prioritize fixes.”

---

### 3. Sysmon — Windows Event Monitoring

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

### Exam Note: Configure Which Events Sysmon Records

As shared in the class group, Sysmon will not record every event category unless its configuration allows those events. Create an XML file with the required configuration, then open **Command Prompt as Administrator**.

![Sysmon configuration example shared for the exam](assets/sysmon-config-example.png)

The example from the class group is:

```xml
<Sysmon schemaversion="4.90">
  <EventFiltering>
    <NetworkConnect onmatch="exclude" />
    <FileCreate onmatch="exclude" />
    <RegistryEvent onmatch="exclude" />
    <DnsQuery onmatch="exclude" />
    <FileDelete onmatch="exclude" />
    <FileDeleteDetected onmatch="exclude" />
  </EventFiltering>
</Sysmon>
```

**Important interpretation:** `onmatch="exclude"` is not an instruction to enable that event category. An empty exclude rule can exclude all events of that category, so this exact example may suppress the network, file, registry and DNS events listed. Use it only if that is the intended filtering behavior; for exam practicals, follow the XML/configuration given by your instructor. The schema version must also be compatible with the installed Sysmon version.

To **apply or update** a configuration on an already installed Sysmon instance, run this in an elevated Command Prompt (replace the filename with your XML filename):

```cmd
Sysmon64.exe -c filename.xml
```

To install Sysmon for the first time with a configuration, use `-i` instead:

```cmd
Sysmon64.exe -accepteula -i filename.xml
```

### Step 3: Install Sysmon with Configuration

Open **Command Prompt as Administrator** and run:

```cmd
cd C:\Users\YourUser\Downloads\Sysmon

Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

**Expected Output:**
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

---

---

### 4. Maltego — OSINT Relationship Mapping

### What is Maltego?
**Maltego** is a powerful OSINT (Open Source Intelligence) and graphical link analysis tool. It performs automated reconnaissance and **visualizes relationships** among domains, email addresses, IP addresses, people, social networks, and infrastructure in an interactive graph.

### Step 1: Open Maltego in Kali Linux

```bash
maltego
```

Or find it in: **Applications → Information Gathering → Maltego**

Maltego comes **pre-installed** in Kali Linux (Community Edition).

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
> - Community Edition is **free** but limited to 12 results per transform
> - It's a **passive** tool — queries public sources, not the target directly

---

---

## KALI LINUX TOOLS

### 5. WHOIS — Domain Registration Lookup

### What is WHOIS?
**WHOIS** is a query protocol used to look up **domain registration information**. It reveals who registered the domain, when, contact details, name servers, and registrar information.

### Step 1: Using WHOIS from Terminal (Kali Linux)

```bash
whois example.com
```

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
> - Works for both domains and IP addresses (`whois 93.184.216.34`)

---

---

### 6. nslookup — DNS Reconnaissance

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

### Step 4: Find All DNS Records (ANY)

```bash
nslookup -type=any example.com
```

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
| **PTR** | Reverse DNS | `nslookup 93.184.216.34` |

> [!TIP]
> **Key Exam Points:**
> - nsLookup queries **DNS servers** for domain information
> - Key record types: **A** (IP), **MX** (mail), **NS** (nameserver), **SOA**, **TXT**
> - Use `-type=<record>` to query specific record types
> - It is a **passive** reconnaissance technique
> - Available on **both Windows and Linux**

---

---

### 7. Nmap — Active Network Reconnaissance

### What is Nmap?
**Nmap (Network Mapper)** is the most widely used **active reconnaissance** tool. It discovers live hosts, open ports, running services, service versions, and operating systems on a target network.

### Step 1: Basic Host Discovery (Ping Scan)

```bash
nmap -sn 192.168.1.0/24
```

**Expected Output:**
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

### Step 5: Aggressive Scan (Most Complete – Recommended for Exam)

```bash
sudo nmap -A target-ip
```

This combines: `-sV` (version) + `-O` (OS) + `-sC` (scripts) + `--traceroute`

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
> - `-A` is the **most comprehensive** single flag — use this in exam
> - Discovers: **live hosts, open ports, services, versions, OS**
> - Use `sudo` for SYN scan and OS detection (requires raw packets)
> - Use `-oN` or `-oA` to save results to files

---
---


These tools were on the exam tool list and are included here as a focused revision section.

---

### 8. Armitage — Metasploit GUI

**Real interface screenshots:** [Armitage GUI walkthrough](https://www.computerweekly.com/tutorial/Metasploit-guide-4-Armitage-cyber-attack-management-GUI) · [Armitage interface paper (PDF)](https://www.usenix.org/system/files/login/issues/login_aug_2011.pdf)

**Purpose:** Armitage is a graphical interface for the Metasploit Framework. It helps a tester visualize discovered hosts, browse modules and manage sessions during an authorized penetration test.

**Startup commands from the lab notes**

```bash
sudo msfdb init
armitage
```

- `sudo msfdb init` initializes the Metasploit database when needed; do not repeatedly initialize an already configured database without a reason.
- `armitage` launches the GUI. Follow the connection dialog to connect to the local Metasploit service.
- Installation and service setup differ between Kali releases; if the command fails, check the installed package and service status rather than assuming the command is universally sufficient.

**Know for the exam**
- **Metasploit:** underlying penetration-testing framework and module collection.
- **Armitage:** GUI that makes Metasploit functionality easier to navigate.
- Typical workflow: connect to the local framework → import/discover authorized lab hosts → inspect services and modules → validate findings → document results.
- Use only intentionally vulnerable lab machines or explicitly authorized targets.

---

### 9. theHarvester — Passive Reconnaissance

### What is theHarvester?
**theHarvester** is an OSINT (Open Source Intelligence) tool used for **passive reconnaissance**. It collects email addresses, subdomains, hostnames, open ports, and banners from public sources like search engines and APIs — **without directly touching the target**.

### Step 1: Open Terminal in Kali Linux

theHarvester comes **pre-installed** in Kali Linux.

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
theHarvester -d example.com -l 200 -b google
```

**Expected Output:**
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

This searches across **all available sources**: google, bing, baidu, linkedin, twitter, virustotal, threatcrowd, etc.

### Step 5: Save Results to an HTML File

```bash
theHarvester -d example.com -l 200 -b google -f results_example
```

This creates `results_example.html` and `results_example.xml` files.

> [!TIP]
> **Key Exam Points:**
> - theHarvester is a **passive** reconnaissance tool (doesn't directly interact with the target)
> - It gathers: **emails, subdomains, hostnames, IPs, banners**
> - Data sources include: Google, Bing, LinkedIn, VirusTotal, etc.
> - Use `-b all` for comprehensive results
> - Use `-f` flag to save output to a file

---

---

### 10. Netdiscover — Local Network Host Discovery

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

### Step 4: Scan a Specific Range

```bash
sudo netdiscover -i eth0 -r 10.0.0.0/16
```

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

---

---

### 11. Sublist3r — Subdomain Enumeration

### What is Sublist3r?
**Sublist3r** is a Python-based tool designed to enumerate **subdomains** of a target domain using passive sources like search engines (Google, Bing, Yahoo, Baidu), VirusTotal, ThreatCrowd, DNSdumpster, and more.

### Step 1: Install Sublist3r (if not installed)

```bash
# Clone from GitHub
git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r

# Install dependencies
pip install -r requirements.txt
```

Or install directly:
```bash
sudo apt install sublist3r
```

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
sublist3r -d example.com -e google,bing,virustotal
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
> - Use `-o` to save results to a file

---
---


---

---

### 12. Wireshark — Packet Capture and Protocol Analysis

![Wireshark logo](https://cdn.simpleicons.org/wireshark/1679A7)

**Real interface screenshots:** [Wireshark official site](https://www.wireshark.org/) · [Wireshark documentation](https://www.wireshark.org/docs/)

**Purpose:** Wireshark captures and analyzes packets travelling through a selected network interface. It is useful for troubleshooting, protocol analysis and incident response.

**Basic workflow**
1. Open Wireshark.
2. Select the correct interface (Ethernet, Wi-Fi, or a lab interface).
3. Start capture and reproduce the activity you need to inspect.
4. Stop capture; select a packet to inspect its protocol layers and fields.
5. Apply a **display filter** to narrow the visible packets.
6. Save the capture as a `.pcapng` file if required.

**Useful display filters**

```text
dns
tcp
udp
http
ip.addr == 192.168.1.10
tcp.port == 443
dns && ip.addr == 192.168.1.10
```

**Important distinction**
- **Capture filter:** limits what is captured, using capture-filter syntax.
- **Display filter:** hides/shows packets after capture; it does not delete hidden packets.

**Know for the exam**
- A packet capture shows observable traffic; encrypted HTTPS payloads generally cannot be read as plain text without the appropriate decryption keys/configuration.
- Capturing on a switched network or Wi-Fi interface does not automatically reveal every other device's traffic.
- Capture only traffic you are authorized to inspect.

---

## BROWSER-BASED TOOLS

### 13. Robtex — DNS and IP Intelligence

**Real lookup example:** [Robtex DNS lookup page](https://robtex.com/en/dns-lookup/com/robtex/dns)

**Purpose:** Robtex is a browser-based lookup service for DNS records, IP addresses, autonomous systems and relationships among internet infrastructure.

**Basic workflow**
1. Open [Robtex](https://www.robtex.com/).
2. Search for an authorized domain, hostname or IP address.
3. Review available DNS records, related hostnames, IP/network details and links between results.
4. Record the source and time because public DNS/intelligence data can change.

**Know for the exam**
- Robtex is an online reconnaissance/intelligence service.
- It is useful for relating domains, IPs and DNS information; it is not a replacement for an active port scanner.
- Results may be incomplete, cached or privacy-redacted.

---

### 14. Shodan — Search Engine for Internet-Connected Devices

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

### Method 2: Using Shodan CLI (Command Line)

```bash
# Install Shodan CLI
pip install shodan

# Initialize with API key (get from shodan.io/dashboard)
shodan init YOUR_API_KEY

# Search for a host
shodan host 93.184.216.34

# Search query
shodan search "hostname:example.com"

# Count results
shodan count "apache country:IN"
```

**Expected CLI Output:**
```
$ shodan host 93.184.216.34

93.184.216.34
City:                    Norwood
Country:                 United States
Organization:            Edgecast Inc.
Updated:                 2026-08-20T10:15:23.456789
Number of open ports:    3

Ports:
     80/tcp  
     443/tcp 
    8080/tcp 
```

### Interesting Shodan Queries for Exam:

```
# Find webcams
webcam country:IN

# Find default passwords
"default password"

# Find specific web servers
product:Apache country:US

# Find open databases
port:27017 (MongoDB)
port:3306 product:MySQL

# Find specific organizations
org:"Target Organization Name"
```

> [!TIP]
> **Key Exam Points:**
> - Shodan indexes **publicly exposed** Internet-connected devices
> - Shows: **open ports, services, versions, banners, SSL certs, vulnerabilities**
> - It is a **passive** tool — Shodan scans the internet; you just query its database
> - Key filters: `hostname:`, `port:`, `org:`, `country:`, `product:`
> - Free account = limited searches; API key needed for CLI

---

---

### 15. Google Dorking — Search-Engine Reconnaissance

### What is Google Dorking?
**Google Dorking** (also called Google Hacking) uses **advanced search operators** in Google to find publicly available but sensitive information about a target — such as login pages, exposed files, directory listings, and more.

### Key Google Dork Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `site:` | Restrict results to a domain | `site:example.com` |
| `intitle:` | Search in page title | `intitle:"index of"` |
| `inurl:` | Search in URL | `inurl:admin` |
| `filetype:` | Search for file types | `filetype:pdf` |
| `intext:` | Search in page body | `intext:"password"` |
| `cache:` | View cached version | `cache:example.com` |
| `ext:` | File extension | `ext:sql` |
| `link:` | Pages linking to a URL | `link:example.com` |
| `-` (minus) | Exclude results | `site:example.com -www` |

### Practical Google Dork Queries for Exam

**1. Find subdomains of a target:**
```
site:example.com -www
```
This finds all indexed subdomains excluding `www`.

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

1. Open **Google Chrome/Firefox** → go to `https://www.google.com`
2. Type the dork query in the search bar
3. Press Enter and analyze the results
4. Document the findings (subdomains, files, pages found)

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
> - Google Dorking is **100% passive** — uses only Google's indexed data
> - No tools needed — just use Google Search
> - Key operators: `site:`, `filetype:`, `intitle:`, `inurl:`, `intext:`
> - **Google Hacking Database (GHDB)** at `exploit-db.com/google-hacking-database` has 1000s of dorks
> - Always use on **authorized targets only**

---
---


---

---

## Optional material not in your listed syllabus


### Optional extra — Nikto web server scanner (not in the supplied tool list)

### What is Nikto?
**Nikto** is an open-source **web server scanner** that checks for dangerous files, outdated server software, version-specific vulnerabilities, and server misconfigurations. It performs **active** scanning.

### Step 1: Basic Scan

```bash
nikto -h http://target-ip
```

or with a domain:

```bash
nikto -h http://example.com
```

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
> - Not stealthy — **generates many log entries** on the target server

---

---


# 📋 Quick Reference – All Tools Summary

| # | Platform | Tool | Key Purpose | Basic Command / Entry Point |
|---:|---|---|---|---|
| 1 | Windows | **FOCA** | Extract document metadata | GUI-based |
| 2 | Windows | **Nessus** | Find vulnerabilities and misconfigurations | Create an authorized scan in the UI |
| 3 | Windows | **Sysmon** | Record process, network and file events | `Sysmon64.exe -accepteula -i config.xml` |
| 4 | Windows | **Maltego** | Map relationships between entities | Launch the Maltego GUI |
| 5 | Kali | **WHOIS** | Domain/IP registration information | `whois example.com` |
| 6 | Kali | **nslookup** | Query DNS records | `nslookup -type=mx example.com` |
| 7 | Kali | **Nmap** | Discover hosts, ports and services | `nmap -sV <authorized-lab-host>` |
| 8 | Kali | **Armitage** | GUI for Metasploit | `sudo msfdb init` then `armitage` |
| 9 | Kali | **theHarvester** | Collect emails, subdomains and hostnames | `theHarvester -d example.com -l 100 -b crtsh` |
| 10 | Kali | **Netdiscover** | ARP-based local host discovery | `sudo netdiscover -r 192.168.1.0/24` |
| 11 | Kali | **Sublist3r** | Discover subdomains | `sublist3r -d example.com` |
| 12 | Kali | **Wireshark** | Inspect packets and protocols | Launch GUI; choose interface |
| 13 | Browser | **Robtex** | DNS/IP/network relationships | Search a domain or IP at [Robtex](https://www.robtex.com/) |
| 14 | Browser | **Shodan** | Search indexed exposed internet services | `hostname:example.com` |
| 15 | Browser | **Google Dorking** | Find indexed pages and files | `site:example.com filetype:pdf` |

---

# 🎯 Exam Strategy Tips

> [!IMPORTANT]
> **Must-Remember for Your Exam:**
> 1. **Read the question carefully** — note whether it says "authorized domain/network"
> 2. **Start with the basic command** — then add flags for more detail
> 3. **Always show and explain the output** — don't just run commands, interpret results
> 4. **Save your outputs** — use `-o` or `-f` flags when available
> 5. **Know Active vs. Passive** — Active tools (Nmap and active Netdiscover) directly interact with the target; public-source tools (theHarvester, WHOIS, Shodan, Robtex, Google Dorking and Sublist3r in passive mode) generally query existing data

> [!WARNING]
> **Key Difference to Remember:**
> - **Passive** = Querying public sources or existing indexes (Google, WHOIS, Shodan, Robtex); this does not normally send scan probes to the target.
> - **Active** = Sending probes or requests directly to the target (Nmap, Nikto, active Netdiscover, Nessus scans); activity may be logged or detected.
> - **Nuance:** classifications depend on mode and data source; do not assume every feature of a tool is passive.

> [!CAUTION]
> **Never run active scans (Nmap, Nessus, or active Netdiscover) on unauthorized targets.** In your exam, only scan lab machines, VMs, or explicitly authorized domains. Unauthorized scanning is **illegal**.

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

*Revision note: prioritize each tool’s purpose, platform, active/passive behavior, basic workflow, command syntax, and interpretation of output.*
