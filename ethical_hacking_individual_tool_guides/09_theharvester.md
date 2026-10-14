# theHarvester — Public-Source Reconnaissance

## 1. Purpose
theHarvester collects publicly available information such as email addresses, hostnames, and subdomains from supported sources. Source coverage and command options change between versions.

**Memory hook:** theHarvester = **collect public breadcrumbs**.

## 2. Inputs
- Domain you own or have permission to assess.
- Source/provider options supported by the installed version.
- Optional result limit.

## 3. Start with help
```bash
theHarvester -h
```
A common command shape in versions that support these options is:
```bash
theHarvester -d example.com -b duckduckgo -l 100
```
Here `-d` is the domain, `-b` selects a source, and `-l` limits results in many versions. Check your local help because flags and source names can differ.

## 4. Output and interpretation
You may see:
- Emails: addresses found in indexed/public sources.
- Hosts: hostnames associated with the domain.
- IP addresses: addresses returned or linked by a source.
- Errors/rate limits: provider may block requests, require an API key, or have changed its response.

Explain: “These are publicly sourced leads, not a complete inventory. I would validate each hostname through DNS and confirm ownership/scope before testing.”

## 5. Common problems
- Unsupported source: run `-h` and use a source name listed by the installed version.
- API key required: configure the source only through its documented setup.
- No results: domain may have little indexed data or the source may restrict access.
- Rate limit: wait or use an authorized alternative source; do not try to bypass access controls.

## 6. Viva
**Q: What is passive reconnaissance?** Gathering information without directly probing the target's services; source interactions may still contact third parties.
**Q: Are results complete?** No.
**Q: Why verify hostnames?** Public-source data may be stale or unrelated.
**Q: Is an email address a vulnerability?** No; it is an information point that must be handled responsibly.
