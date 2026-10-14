# Ethical Hacking Lab Master Guide

## Tool explanations, commands, GUI workflows, viva questions, and memory cues

**Lab context:** Windows host + Oracle VirtualBox + Kali Linux + an
isolated Metasploitable 2 target.\
**Purpose:** Learn defensive security, reconnaissance, network
discovery, vulnerability assessment, and traffic/system analysis in an
authorized lab.

> **Safety rule:** Only scan systems you own or have explicit permission
> to assess. Keep Metasploitable 2 on a host-only or otherwise isolated
> lab network. Do not expose it through bridged networking or port
> forwarding. Commands and menus may vary by software version.

------------------------------------------------------------------------

# 1. The big picture

A typical assessment workflow is:

1.  **Plan and authorize** --- define scope, targets, time window, and
    permitted tests.
2.  **Reconnaissance** --- gather public information using WHOIS, DNS,
    Robtex, Shodan, theHarvester, Sublist3r, FOCA, and Google search
    operators.
3.  **Host and service discovery** --- identify lab hosts and services
    with Netdiscover and Nmap.
4.  **Vulnerability assessment** --- use Nessus to identify possible
    weaknesses and verify findings responsibly.
5.  **Controlled validation** --- use Metasploit/Armitage only in a
    deliberately vulnerable lab and only within scope.
6.  **Observe and investigate** --- use Sysmon for Windows events,
    Wireshark for packets, and Maltego to map relationships.
7.  **Report** --- record evidence, impact, confidence, and remediation.

Reconnaissance is information gathering; scanning actively probes
systems; vulnerability assessment checks for weaknesses; monitoring
records activity. They are related but not interchangeable.

## Quick tool map

  Tool                   One-line purpose
  ---------------------- -----------------------------------------------------
  VirtualBox             Runs virtual machines
  Kali Linux             Security-testing operating system
  Metasploitable 2       Intentionally vulnerable practice target
  WHOIS                  Registration and allocation records
  nslookup               DNS queries
  Nmap                   Host, port, and service discovery
  Netdiscover            Local-network host discovery using ARP
  theHarvester           Public-source domain, host, and email discovery
  Sublist3r              Candidate subdomain enumeration
  Robtex                 Online DNS/IP relationship lookup
  Shodan                 Search engine for observed internet-facing services
  Google Dorking         Targeted search using search operators
  FOCA                   Document metadata analysis
  Nessus                 Vulnerability scanning
  Sysmon                 Detailed Windows event logging
  Maltego                Relationship graphing
  Wireshark              Packet capture and protocol analysis
  Metasploit Framework   Security testing framework
  Armitage               Older graphical interface for Metasploit

------------------------------------------------------------------------

# 2. Safe lab setup: VirtualBox, Kali, and Metasploitable 2

## What each component does

-   **Windows host:** Your normal computer and the system running
    VirtualBox.
-   **VirtualBox:** Creates isolated virtual computers and virtual
    networks.
-   **Kali Linux:** The tester machine, where tools such as Nmap and
    Metasploit run.
-   **Metasploitable 2:** A deliberately vulnerable Linux VM designed
    for security training.
-   **Host-only network:** Allows the host and attached VMs to
    communicate on a private virtual network.
-   **NAT adapter:** Usually gives Kali outbound internet access for
    updates. The vulnerable target does not need this adapter.

## Recommended network design

-   Kali Adapter 1: NAT (for package updates).
-   Kali Adapter 2: Host-only network (for the lab).
-   Metasploitable 2 Adapter 1: The same host-only network only.
-   Do not use bridged networking for the vulnerable target.
-   Do not create router port-forwarding rules to expose the target.

Example addresses such as `192.168.56.1` and `192.168.56.102` are
examples only. Use the subnet shown in your own VirtualBox Network
Manager.

## Setup checklist

1.  Download Metasploitable 2 from the official Rapid7/SourceForge
    project page.
2.  Extract the downloaded archive.
3.  Create/import a VM that uses the supplied virtual disk rather than a
    new blank disk.
4.  Set the target's network adapter to the same host-only network as
    Kali's lab adapter.
5.  Boot both VMs and identify their IP addresses.
6.  Test connectivity and scan only the target VM.

Useful Kali commands:

``` bash
ip -br addr
ip route
ping -c 4 TARGET_IP
nmap -sn LAB_SUBNET/CIDR
```

Replace placeholders with values from your own lab. A failed ping does
not prove that a host is offline; ICMP may be blocked.

------------------------------------------------------------------------

# 3. WHOIS

## What it does

WHOIS queries registration or IP-allocation databases. Depending on the
registry and privacy settings, results may include a registrar,
registration dates, nameservers, organization, or network allocation.

## Commands

``` bash
whois example.com
whois 8.8.8.8
```

Use a domain you own or are allowed to investigate. The IP example is a
public DNS resolver and is shown only as a syntax example.

## How to read the output

-   **Registrar:** Company through which a domain is registered.
-   **Creation/expiration dates:** Registration lifecycle details when
    available.
-   **Name Server:** Nameservers associated with the domain.
-   **Organization / NetRange / CIDR:** Allocation details for an IP
    range.
-   **Redacted fields:** Privacy or registry policy may hide contact
    details.

WHOIS data does not necessarily identify the web host. A registrar is
not automatically the hosting provider, and an IP allocation may belong
to a cloud provider or ISP.

## GUI

WHOIS is mainly a command-line tool. Online lookup sites provide a
browser-based alternative. Cross-check important details with the
registry or official source.

## Common errors

-   `whois: command not found`: install the package if available for
    your Kali release.
-   Incomplete output: the registry may limit or redact data.
-   Unexpected organization: the address may belong to a CDN, cloud
    provider, or ISP.

## Viva questions

1.  **What is WHOIS?** A query mechanism for domain registration and IP
    allocation records.
2.  **What is a registrar?** A service through which a domain name is
    registered.
3.  **Does WHOIS reveal the physical server?** Not necessarily.
4.  **Why might owner details be hidden?** Privacy protection and
    registry policies.
5.  **How is WHOIS different from DNS?** WHOIS provides
    registration/allocation information; DNS resolves names and
    publishes records.

**Memory cue:** WHOIS = **Who registered/was allocated this?**

------------------------------------------------------------------------

# 4. nslookup

## What it does

`nslookup` sends DNS queries to a resolver. It can look up addresses and
record types such as A, AAAA, MX, NS, and TXT.

## Commands

``` bash
nslookup example.com
nslookup -type=A example.com
nslookup -type=AAAA example.com
nslookup -type=MX example.com
nslookup -type=NS example.com
nslookup -type=TXT example.com
```

Interactive mode:

``` text
nslookup
set type=MX
example.com
exit
```

## Record types to memorize

  Record   Meaning
  -------- -----------------------------------------------------------
  A        Name to IPv4 address
  AAAA     Name to IPv6 address
  MX       Mail exchanger
  NS       Authoritative nameserver
  TXT      Text data, often used for verification and email policies
  CNAME    Alias to another name
  PTR      Reverse lookup from an IP address to a name

## GUI

You can use a browser-based DNS lookup site, but terminal output is
useful in a lab. In output, distinguish the resolver's address from the
answer records for the queried name.

## Common errors

-   `NXDOMAIN`: the queried name does not exist according to the
    responding DNS system.
-   Timeout: resolver/network problems or filtering may be involved.
-   No MX record: does not automatically mean the domain cannot send
    email; it may simply not receive mail directly.
-   Different answers: DNS responses can vary by resolver, location,
    caching, and time.

## Viva questions

1.  **What is DNS?** The system that maps names to records such as IP
    addresses.
2.  **What does an A record contain?** An IPv4 address.
3.  **What is an MX record?** A mail-exchanger record.
4.  **What does NXDOMAIN mean?** The queried name does not exist
    according to the resolver.
5.  **Why can two resolvers return different answers?** Caching,
    geography, split DNS, or changes in records.

**Memory cue:** `nslookup` = **Name-server lookup**.

------------------------------------------------------------------------

# 5. Nmap

## What it does

Nmap discovers hosts and examines ports and services. Service/version
detection can help identify software that needs maintenance.

## Safe lab commands

``` bash
nmap TARGET_IP
nmap -sn LAB_SUBNET/CIDR
nmap -sV TARGET_IP
nmap -p 22,80,443 TARGET_IP
nmap -oN scan.txt TARGET_IP
nmap -oX scan.xml TARGET_IP
```

-   Default scan: checks common ports on the target.
-   `-sn`: host discovery without a port scan.
-   `-sV`: attempts service/version detection.
-   `-p`: selects ports.
-   `-oN`: writes normal text output.
-   `-oX`: writes XML output.

Only use the commands against your own lab or authorized scope. Do not
use broad scans against public networks without explicit permission.

## Read the results

  -----------------------------------------------------------------------
  State                               Meaning
  ----------------------------------- -----------------------------------
  open                                A service is accepting connections
                                      on that port

  closed                              The host is reachable, but no
                                      service is listening there

  filtered                            Filtering or network behavior
                                      prevents a confident state
                                      determination

  SERVICE                             A conventional service label; not
                                      always a confirmed identity
  -----------------------------------------------------------------------

An open port is not automatically a vulnerability. A service name is not
always proof of the exact software/version.

## GUI: Zenmap

Zenmap is Nmap's graphical interface when available.

1.  Enter the lab IP in **Target**.
2.  Select a suitable profile or enter an authorized command.
3.  Click **Scan**.
4.  Review the output and host/port views.
5.  Save the results for your lab report.

Zenmap may not be installed by default in every Kali release. Nmap works
from the terminal without it.

## Common errors

-   `Host seems down`: host may be offline or may block discovery
    probes; verify the IP and lab network.
-   All ports filtered: a firewall or network path may be blocking
    probes.
-   Version not detected: the service may not provide enough
    information.
-   Wrong subnet: check `ip -br addr` and `ip route`.

## Viva questions

1.  **What is Nmap?** A network discovery and security auditing tool.
2.  **What does `-sn` do?** Host discovery without a port scan.
3.  **What does `-sV` do?** Attempts to identify service versions.
4.  **What is an open port?** A port accepting connections.
5.  **Is an open port a vulnerability?** No; risk depends on service,
    configuration, exposure, and context.
6.  **What does filtered mean?** Nmap cannot determine the state
    reliably because probes are blocked or affected.
7.  **Why save XML output?** It can be processed by other tools and
    reporting workflows.

**Memory cue:** Nmap = **Network map: hosts → ports → services**.

------------------------------------------------------------------------

# 6. Netdiscover

## What it does

Netdiscover identifies devices on a local IPv4 network, commonly using
ARP. It is useful on the same Layer-2 network segment as your lab VMs.

## Commands

``` bash
ip -br addr
ip route
sudo netdiscover
sudo netdiscover -i eth0
sudo netdiscover -r 192.168.56.0/24
```

The interface and subnet are examples. Check the actual interface name
and range first.

## How to interpret results

-   **IP address:** Local address observed.
-   **MAC address:** Layer-2 hardware address reported for the device.
-   **Vendor:** May be inferred from the MAC address prefix; virtual
    NICs can have virtualized vendor identifiers.
-   **Response:** Indicates a device responded to discovery traffic.

## GUI

Netdiscover is terminal-oriented. Kali's network settings can help
inspect interfaces, but they are not a direct replacement for its
discovery results.

## Limitations

-   ARP discovery normally works only on the local network segment.
-   It does not discover arbitrary remote internet hosts through
    routers.
-   A host may not appear if it is offline, on another subnet, or not
    responding.

## Viva questions

1.  **What is ARP?** Address Resolution Protocol; it resolves local IPv4
    addresses to link-layer addresses.
2.  **Why use Netdiscover?** To discover hosts on a local network.
3.  **Does ARP cross routers?** Normally no; routers separate Layer-2
    broadcast domains.
4.  **What is a MAC address?** A link-layer address used on a local
    network.
5.  **How is Netdiscover different from Nmap?** Netdiscover focuses on
    local host discovery via ARP; Nmap supports broader host, port, and
    service discovery.

**Memory cue:** Netdiscover = **Nearby network devices**.

------------------------------------------------------------------------

# 7. theHarvester

## What it does

theHarvester collects publicly available information from supported
sources, such as email addresses, hostnames, and subdomains. Sources,
flags, and authentication requirements vary by version.

## Start by checking the installed options

``` bash
theHarvester -h
```

A source-specific example, if supported by the installed version:

``` bash
theHarvester -d example.com -b crtsh
```

Use only domains in your authorized scope. Check the help output for
currently supported sources rather than assuming old tutorials are
accurate.

## GUI

theHarvester is primarily command-line based. Its results can be
reviewed in the terminal and, depending on version, exported to a file.

## How to interpret results

-   **Email:** Publicly observed address; not proof the account is
    active.
-   **Host:** A candidate hostname; not proof the host is live.
-   **Source:** Helps trace where the result came from.
-   **No results:** Could mean source limitations, API/authentication
    requirements, rate limits, or genuinely little indexed information.

## Viva questions

1.  **What is theHarvester used for?** Public-source reconnaissance.
2.  **Does a discovered email prove an account exists?** No.
3.  **Why might a source fail?** Provider changes, quotas, network
    issues, or missing credentials.
4.  **What is passive reconnaissance?** Gathering information without
    directly probing the target's systems.
5.  **Why record sources?** To support verification and evidence
    quality.

**Memory cue:** theHarvester = **Harvest public clues**.

------------------------------------------------------------------------

# 8. Sublist3r

## What it does

Sublist3r enumerates candidate subdomains using supported public sources
and, in some versions, brute-force techniques. It is an older project,
so individual sources may stop working.

## Commands

``` powershell
python sublist3r.py -h
python sublist3r.py -d example.com
python sublist3r.py -d example.com -o subdomains.txt
python sublist3r.py -d example.com -b
```

These are examples for a copy of Sublist3r that supports those options.
Check `-h` because flags can vary. Run from the directory containing
`sublist3r.py`.

## Understanding a common error

A traceback such as:

``` python
token = csrf_regex.findall(resp)[0]
IndexError: list index out of range
```

means the regular expression found no matching token. The website may
have changed, returned a block page, or produced a different response.
It is not proof that no subdomains exist.

Installing optional color libraries may remove a color warning, but it
does not fix a failed data-source scraper.

## Workflow

1.  Check the tool's help.
2.  Run it against a permitted domain.
3.  Save the results.
4.  Validate candidates using DNS queries.
5.  Note source errors and do not treat missing results as proof of
    absence.

## Viva questions

1.  **What is a subdomain?** A name below another domain, such as
    `docs.example.com`.
2.  **What is subdomain enumeration?** Finding candidate names
    associated with a domain.
3.  **Does a discovered name guarantee a live host?** No.
4.  **Why might Sublist3r fail?** Older scrapers may not match current
    provider responses.
5.  **What should follow enumeration?** Authorized DNS validation and
    asset verification.

**Memory cue:** Sublist3r = **Subdomain list**.

------------------------------------------------------------------------

# 9. Wireshark

## What it does

Wireshark captures and decodes packets so you can inspect DNS, TCP, UDP,
HTTP, ICMP, and many other protocols. The amount of application content
visible depends on encryption and capture location.

## GUI workflow

1.  Open Wireshark.
2.  Select the interface carrying your lab traffic.
3.  Start capture.
4.  Generate permitted traffic, such as a DNS lookup from Kali.
5.  Stop capture.
6.  Enter a display filter.
7.  Select a packet and expand its protocol fields.
8.  Save the capture as a `.pcapng` file if needed.

## Useful display filters

  Filter                        Meaning
  ----------------------------- ------------------------------------------------
  `dns`                         DNS packets
  `tcp`                         TCP packets
  `udp`                         UDP packets
  `http`                        Recognized HTTP packets
  `icmp`                        ICMP packets
  `ip.addr == 192.168.56.102`   IPv4 packets involving that address
  `tcp.port == 80`              TCP packets with source or destination port 80

Display filters are applied to captured packets. Capture filters use a
different syntax.

## Terminal alternatives

``` bash
tshark -D
tshark -i eth0
tshark -r capture.pcapng
```

Check interface names and permissions. Capturing may require elevated
privileges or configured capture permissions.

## What to inspect

-   Source and destination addresses
-   Protocol and ports
-   DNS query name and answer
-   TCP handshake flags
-   Retransmissions and timing
-   Whether traffic is encrypted

HTTPS/TLS generally protects application content. Packet metadata may
remain visible, but you should not expect to read encrypted page
contents from a normal capture.

## Viva questions

1.  **What is Wireshark?** A packet analyzer.
2.  **What is a packet?** A unit of data carried by a network protocol.
3.  **What is a display filter?** A filter that controls which captured
    packets are shown.
4.  **What is the difference between TCP and UDP?** TCP provides a
    connection-oriented reliable byte stream; UDP sends datagrams
    without TCP's delivery guarantees.
5.  **Why might HTTPS content not be readable?** The application data is
    encrypted.
6.  **What is a PCAP/PCAPNG file?** A file format for packet-capture
    data.
7.  **Does seeing a packet mean the whole conversation is captured?**
    No; capture interface, filters, packet loss, and encryption affect
    visibility.

**Memory cue:** Wireshark = **Watch the wire**.

------------------------------------------------------------------------

# 10. Metasploit Framework

## What it does

Metasploit is a security-testing framework that organizes modules for
tasks such as auxiliary checks, exploit validation in authorized labs,
payload handling, and post-exploitation testing.

## Basic console commands

``` bash
msfconsole
```

Inside the console:

``` text
help
version
search type:auxiliary
exit
```

These show help, version, search for auxiliary modules, and exit.

Module types commonly include:

-   **Auxiliary:** Supporting tasks such as scanning or service checks.
-   **Exploit:** Code intended to trigger a vulnerability.
-   **Payload:** Code executed as a consequence of a successful exploit.
-   **Post:** Modules for authorized post-exploitation assessment.
-   **Encoder / NOP:** Specialized module types used in certain payload
    workflows.

Module availability and behavior depend on the installed version.

## Database

``` bash
sudo msfdb status
sudo msfdb init
```

Database setup varies by distribution/version. Initialize only if
needed, and inspect the command's output. The database can store hosts,
services, and other assessment data.

## Safety

Practice exploitation only on deliberately vulnerable VMs you control.
Before running any module, understand its purpose, target requirements,
options, side effects, and scope. Do not use public systems as practice
targets.

## Viva questions

1.  **What is Metasploit?** A modular security-testing framework.
2.  **What is an exploit module?** A module designed to trigger a
    vulnerability under specified conditions.
3.  **What is a payload?** Code or functionality delivered/executed as
    part of an exploit workflow.
4.  **What is an auxiliary module?** A supporting module for tasks such
    as checks or scanning.
5.  **Why use a database?** To store and organize assessment
    information.
6.  **Does a successful module run prove a vulnerability?** Not always;
    understand the module's evidence and validate safely.

**Memory cue:** Metasploit = **Modular security testing**.

------------------------------------------------------------------------

# 11. Armitage

## What it does

Armitage is an older graphical interface for Metasploit. It helps
organize hosts, browse modules, and interact with the Metasploit
service. Compatibility may be limited on modern systems.

## Launch and basic checks

``` bash
which armitage
armitage
msfconsole --version
sudo msfdb status
```

If Armitage is not installed and your Kali repository provides it:

``` bash
sudo apt update
sudo apt install armitage metasploit-framework
```

If the package is unavailable, don't download random repackaged
binaries. Check the official Kali package page and your current release.

## GUI areas

-   **Hosts area:** Displays discovered machines and their details.
-   **Modules area:** Lists available modules and their
    descriptions/options.
-   **Console area:** Shows messages and allows interaction with the
    framework.
-   **Host/context menus:** Offer actions depending on the version and
    selected entity.

## Beginner workflow

1.  Start the isolated lab VMs.
2.  Confirm the target IP and subnet.
3.  Launch Armitage and connect to the local Metasploit service.
4.  Discover or import information about the lab host using supported
    features.
5.  Review hosts and service data.
6.  Compare the information with Nmap output.
7.  Practice reading module descriptions before considering any
    validation action.

## Troubleshooting

-   `command not found`: package is not installed or not on PATH.
-   Connection/database error: check the error text, Metasploit version,
    database status, and whether the framework service expected by
    Armitage is running.
-   GUI fails to start: compatibility or Java/runtime dependencies may
    be involved.
-   No hosts shown: the lab may not have been discovered/imported, or
    networking may be misconfigured.

## Viva questions

1.  **What is Armitage?** A graphical interface for Metasploit.
2.  **Does Armitage replace Metasploit?** No; it provides a GUI for
    interacting with the framework.
3.  **Why might it fail on a current Kali release?** It is an older
    project and compatibility varies.
4.  **What is a host view?** A visual representation of discovered
    systems.
5.  **What should you do before using a module?** Read its description,
    requirements, options, impact, and scope.

**Memory cue:** Armitage = **Metasploit's GUI**.

------------------------------------------------------------------------

# 12. FOCA

## What it does

FOCA analyzes documents and metadata. Depending on version and workflow,
it can help identify author/creator fields, software details,
timestamps, and other document properties.

## GUI workflow

1.  Create a project.
2.  Specify an authorized domain or work with documents already
    obtained.
3.  Search for supported document types if using its discovery features.
4.  Download only documents you are permitted to analyze.
5.  Analyze document metadata.
6.  Review and record relevant properties.
7.  Save the project or export findings where supported.

FOCA is primarily GUI-driven rather than a general-purpose command-line
utility.

## Metadata to recognize

-   Author and creator
-   Creation and modification times
-   Office/PDF software
-   Document properties
-   Embedded names or internal paths, when present

Metadata can be incomplete, inaccurate, or deliberately modified. It
should be treated as a clue, not proof of identity.

## Viva questions

1.  **What is metadata?** Data describing other data.
2.  **What does FOCA analyze?** Document metadata and related publicly
    available document information.
3.  **Why can metadata matter?** It can reveal unnecessary information
    about document creation or internal workflows.
4.  **Does an author field prove who wrote a file?** No.
5.  **How can an organization reduce metadata exposure?** Review and
    sanitize document properties before public release.

**Memory cue:** FOCA = **File/document metadata**.

------------------------------------------------------------------------

# 13. Nessus

## What it does

Nessus scans authorized hosts for potential vulnerabilities, outdated
software, insecure settings, and other findings.

## GUI workflow

1.  Start the Nessus service.
2.  Open the configured web interface, commonly
    `https://localhost:8834`.
3.  Sign in.
4.  Choose **Scans → New Scan**.
5.  Select a scan template suitable for the lab.
6.  Enter only approved target IPs.
7.  Save and launch the scan.
8.  Review hosts, findings, severity, evidence, and remediation
    guidance.
9.  Export results if required for your lab report.

## PowerShell service check

``` powershell
Get-Service *nessus*
```

The service name and management commands vary by installation.

## Severity

-   **Critical:** Potentially severe exposure.
-   **High:** Significant risk requiring prompt review.
-   **Medium:** Meaningful weakness to assess.
-   **Low:** Lower-impact issue.
-   **Informational:** Context or configuration details.

Severity is a prioritization aid. A scanner result may be a false
positive or require validation.

## What to record

For each finding, note:

1.  Host and port.
2.  Plugin/finding name.
3.  Evidence.
4.  Severity and rationale.
5.  Recommended remediation.
6.  Whether the result was validated.

## Viva questions

1.  **What is Nessus?** A vulnerability scanner.
2.  **What is a false positive?** A finding reported when the
    vulnerability is not actually present.
3.  **Is a vulnerability scan the same as a penetration test?** No;
    scanning identifies possible weaknesses, while a penetration test
    includes controlled validation within scope.
4.  **Why use credentials for some scans?** Authenticated checks can
    inspect patch/configuration details more accurately.
5.  **How should findings be prioritized?** Consider severity,
    exploitability, exposure, asset importance, and business impact.

**Memory cue:** Nessus = **Find weaknesses**.

------------------------------------------------------------------------

# 14. Sysmon

## What it does

Sysmon is a Windows system-monitoring service and driver that records
detailed events. Logs are commonly inspected in Windows Event Viewer or
forwarded to a log-analysis platform.

## Installation and basic commands

Open PowerShell as Administrator in the directory containing the Sysmon
executable.

``` powershell
.\sysmon64.exe -accepteula -i
.\sysmon64.exe -c
.\sysmon64.exe -s
```

-   `-i`: installs Sysmon.
-   `-c`: displays or updates configuration, depending on supplied
    arguments.
-   `-s`: displays the configuration schema.

To apply a configuration file that you have reviewed:

``` powershell
.\sysmon64.exe -c .\sysmonconfig.xml
```

The file must exist, and the configuration should be appropriate for
your lab.

Uninstalling:

``` powershell
.\sysmon64.exe -u
```

This removes the service and driver. It does not necessarily erase
historical events already written to Windows logs.

## GUI workflow

1.  Open Event Viewer (`eventvwr.msc`).
2.  Navigate to **Applications and Services Logs → Microsoft → Windows →
    Sysmon → Operational**.
3.  Open an event.
4.  Inspect General and Details.
5.  Use **Filter Current Log** to filter by event ID.

## Event IDs to memorize

    ID Event
  ---- -----------------------------------
     1 Process creation
     3 Network connection, when enabled
     7 Image/DLL loaded, when configured
    11 File created
    22 DNS query

Not every event type is enabled by every configuration.

## Safe test

``` powershell
Start-Process notepad.exe
```

Look for Event ID 1 and inspect process image, command line, parent
image, and process ID.

## Viva questions

1.  **What is Sysmon?** A Windows system-monitoring tool that records
    detailed event data.
2.  **Where are Sysmon events viewed?** Event Viewer or a connected
    log-analysis system.
3.  **What is Event ID 1?** Process creation.
4.  **Does Sysmon block malware?** No; it records activity and supports
    detection/investigation.
5.  **Why use a configuration file?** To control event collection and
    manage noise/performance.
6.  **Does uninstalling Sysmon delete old events?** Not necessarily.

**Memory cue:** Sysmon = **System activity logs**.

------------------------------------------------------------------------

# 15. Maltego

## What it does

Maltego visualizes relationships between entities such as domains, IP
addresses, organizations, and email addresses. A transform takes an
entity and retrieves related data from a supported source.

## GUI workflow

1.  Open Maltego Graph and create a graph.
2.  Add an entity, such as a Domain.
3.  Enter an authorized domain.
4.  Open the entity's context menu.
5.  Browse available transforms.
6.  Run a suitable passive information-gathering transform.
7.  Inspect the returned entities and links.
8.  Save/export the graph as supported by your edition.

## Example conceptual graph

`Domain → DNS record → IP address → ASN/organization`

A graph edge indicates a reported relationship, not necessarily
ownership or causation. Transform availability depends on the installed
edition, provider, credentials, and quotas.

## Viva questions

1.  **What is Maltego?** A visual link-analysis and OSINT tool.
2.  **What is an entity?** An object such as a domain, IP, or
    organization.
3.  **What is a transform?** An operation that retrieves related data
    for an entity.
4.  **Does a graph prove ownership?** No; relationships require context
    and verification.
5.  **Why use a graph instead of a list?** It helps reveal connections
    and investigation paths.

**Memory cue:** Maltego = **Map relationships**.

------------------------------------------------------------------------

# 16. Robtex

## What it does

Robtex is a browser-based service for exploring DNS and IP
relationships.

## GUI workflow

1.  Open https://www.robtex.com/.
2.  Search an authorized domain or IP address.
3.  Inspect available DNS records and IP-related information.
4.  Follow relevant relationships.
5.  Verify important results with another source or with `nslookup`.

## What to remember

-   A/AAAA: address records.
-   MX: mail exchangers.
-   NS: nameservers.
-   PTR: reverse DNS name.
-   Shared IPs may host multiple domains.
-   Online data may be cached or incomplete.

Robtex is a website; it does not require a local terminal command.

## Viva questions

1.  **What is Robtex used for?** DNS and IP intelligence.
2.  **Can one IP host many domains?** Yes, for example with shared
    hosting or CDNs.
3.  **Why cross-check its results?** Data can be cached, incomplete, or
    differ by resolver.
4.  **Is Robtex a port scanner?** Its primary role is
    relationship/intelligence lookup, not active port scanning.

**Memory cue:** Robtex = **Relationship lookup**.

------------------------------------------------------------------------

# 17. Shodan

## What it does

Shodan searches collected observations about internet-facing devices and
services. Records can include IPs, ports, banners, and product/version
clues.

Website: https://www.shodan.io/

## GUI workflow

1.  Open Shodan.
2.  Enter a search query.
3.  Open a result to review observed ports and service banners.
4.  Check observation timestamps and context.
5.  Record relevant findings for authorized assets.
6.  Verify important findings against your own asset inventory.

## Common search filters

``` text
hostname:example.com
port:443
country:IN
org:"Example Organization"
net:192.0.2.0/24
```

These are syntax examples. The reserved `192.0.2.0/24` range is used for
documentation and may return no real hosts. Search results depend on
Shodan's current index and account access.

## Interpretation

-   A result reflects a collected observation, not necessarily the
    current state.
-   A port observation does not prove a vulnerability.
-   A product banner can be inaccurate, incomplete, or out of date.
-   Use Shodan to inspect authorized assets, not as permission to probe
    or access other people's systems.

## Viva questions

1.  **What is Shodan?** A search engine for observed internet-connected
    services/devices.
2.  **How is it different from Nmap?** Shodan searches previously
    collected observations; Nmap actively probes a target.
3.  **What is a banner?** Information a service exposes about itself.
4.  **Does a Shodan result prove a host is currently online?** No.
5.  **What is a search filter?** A field that narrows the result set.

**Memory cue:** Shodan = **Search exposed services**.

------------------------------------------------------------------------

# 18. Google Dorking

## What it does

Google Dorking uses advanced search operators to narrow public search
results. It is not a separate application.

## Common operators

  ------------------------------------------------------------------------------------------
  Operator                Example                                    Purpose
  ----------------------- ------------------------------------------ -----------------------
  `site:`                 `site:example.com`                         Limit to a domain/site

  `filetype:`             `site:example.com filetype:pdf`            Find indexed PDFs

  `intitle:`              `site:example.com intitle:documentation`   Match terms in page
                                                                     titles

  `inurl:`                `site:example.com inurl:docs`              Match terms in URLs

  `"exact phrase"`        `site:example.com "security policy"`       Search an exact phrase

  `-term`                 `site:example.com -jobs`                   Exclude a term
  ------------------------------------------------------------------------------------------

## Workflow

1.  Open Google.
2.  Enter a narrow query for an authorized domain.
3.  Review titles, URLs, and snippets.
4.  Open only material you are permitted to access.
5.  Record the query and source.
6.  Verify important claims against the original source.

Do not use search operators to hunt for third-party passwords, private
keys, confidential files, or other sensitive material. For training, use
sample documents or instructor-provided datasets.

## Viva questions

1.  **What is Google Dorking?** Using advanced search operators to
    narrow results.
2.  **What does `site:` do?** Restricts results to a site/domain.
3.  **What does `filetype:pdf` do?** Narrows results to indexed PDF
    files.
4.  **Does an indexed page mean it is authorized to access?** No;
    indexing does not grant permission.
5.  **Can search results be incomplete?** Yes; indexing and ranking
    change over time.

**Memory cue:** Google Dorking = **Targeted searching**.

------------------------------------------------------------------------

# 19. Viva quick-fire revision

Try answering each question aloud before reading the answer.

1.  **Reconnaissance vs scanning?** Reconnaissance gathers information;
    scanning actively probes systems or services.
2.  **WHOIS vs DNS?** Registration/allocation records vs DNS
    records/resolution.
3.  **A vs AAAA?** IPv4 vs IPv6 address record.
4.  **Nmap `-sn`?** Host discovery without port scanning.
5.  **Nmap `-sV`?** Attempts service/version detection.
6.  **Open vs filtered port?** Open accepts connections; filtered cannot
    be reliably classified due to filtering/network behavior.
7.  **Netdiscover's main protocol?** ARP.
8.  **theHarvester's role?** Public-source information gathering.
9.  **Why can Sublist3r throw an IndexError?** A scraper expected a
    match, but received none.
10. **Wireshark display filter?** Filters which captured packets are
    displayed.
11. **Why can't HTTPS content usually be read in a capture?**
    Application data is encrypted.
12. **Nessus vs Nmap?** Vulnerability assessment vs host/port/service
    discovery, though their capabilities can overlap.
13. **Sysmon Event ID 1?** Process creation.
14. **FOCA?** Document metadata analysis.
15. **Maltego transform?** Operation that finds data related to an
    entity.
16. **Shodan vs Nmap?** Collected internet observations vs active
    probing.
17. **Armitage?** Older graphical interface for Metasploit.
18. **Metasploit payload?** Code/functionality associated with an
    exploit workflow.
19. **What is a false positive?** A reported issue that is not actually
    present.
20. **Why isolate Metasploitable 2?** It is deliberately vulnerable and
    must not be exposed to untrusted networks.

------------------------------------------------------------------------

# 20. Memory palace: remember the entire toolkit

Group tools by the question they answer:

## "Who and what is publicly visible?"

-   WHOIS --- registration
-   nslookup --- DNS
-   Robtex --- DNS/IP relationships
-   Shodan --- exposed services
-   Google Dorking --- indexed public pages
-   theHarvester --- public-source clues
-   Sublist3r --- subdomains
-   FOCA --- document metadata
-   Maltego --- relationship graph

## "What is on my lab network?"

-   Netdiscover --- local devices
-   Nmap --- ports and services

## "What weaknesses might exist?"

-   Nessus --- vulnerability findings

## "What happened on the machine or network?"

-   Sysmon --- Windows events
-   Wireshark --- packets

## "How do I manage a controlled validation lab?"

-   VirtualBox --- virtual machines/network
-   Kali --- testing workstation
-   Metasploitable 2 --- vulnerable target
-   Metasploit --- testing framework
-   Armitage --- Metasploit GUI

**Mnemonic:** **Find → Map → Scan → Assess → Observe → Report.**

------------------------------------------------------------------------

# 21. Practical lab report template

Use this template for each exercise.

``` text
Experiment title:
Date:
Authorization/scope:
Tool and version:
Tester machine:
Target lab IP:
Network/interface:
Objective:
Commands or GUI steps:
Observed results:
Evidence file/screenshot:
Interpretation:
Limitations or errors:
Risk/impact:
Recommended remediation:
Conclusion:
```

## Evidence and reporting tips

-   Record tool versions and timestamps.
-   Keep raw output where permitted.
-   Separate observations from assumptions.
-   Document failed scans and source errors honestly.
-   Don't report an open port as a vulnerability without supporting
    evidence.
-   Redact secrets and personal information from screenshots.
-   Include remediation, not just tool output.

------------------------------------------------------------------------

# 22. Official references

-   VirtualBox manual: https://www.virtualbox.org/manual/
-   Kali tools: https://www.kali.org/tools/
-   Nmap reference: https://nmap.org/book/man.html
-   Metasploit documentation: https://docs.metasploit.com/
-   Metasploitable 2: https://sourceforge.net/projects/metasploitable/
-   Nessus documentation: https://docs.tenable.com/nessus/
-   Sysmon:
    https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
-   Wireshark: https://www.wireshark.org/docs/
-   Maltego: https://docs.maltego.com/
-   FOCA: https://github.com/ElevenPaths/FOCA
-   Shodan: https://www.shodan.io/
-   Robtex: https://www.robtex.com/
-   OWASP Juice Shop (another legal training target):
    https://owasp.org/www-project-juice-shop/

**Final study advice:** For each tool, memorize four things: **purpose,
one basic command or GUI path, how to interpret one result, and one
limitation**. That is a strong foundation for practical exams and viva
questions.
