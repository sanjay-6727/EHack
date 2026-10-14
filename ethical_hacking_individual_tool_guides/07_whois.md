# WHOIS — Domain/IP Registration Lookup

## 1. Purpose
WHOIS queries registration-related records for a domain or IP range. Availability and fields depend on the registry, registrar, privacy settings, and RDAP migration.

**Memory hook:** WHOIS asks **who registered this internet name/range?**

## 2. Inputs
A domain such as `example.com` or an IP address. Use domains you own or public test domains for practice.

## 3. Commands
```bash
whois example.com
whois 8.8.8.8
```
If the command is missing on Kali:
```bash
sudo apt update
sudo apt install whois
```

## 4. Typical output fields
- Registrar/registry: organization responsible for registration record.
- Creation/updated/expiry dates: lifecycle metadata when published.
- Name servers: DNS servers listed for the domain.
- Registrant/organization: may be redacted for privacy.
- NetRange/CIDR/OrgName for IP records: allocation information, often about a provider or organization.

## 5. Explain the output
“The WHOIS result shows registration/allocation metadata, such as the registrar, dates, or responsible network organization. It does not reveal every person using the domain or prove that the domain is malicious.”

## 6. Common issues
- No result or referral: try the correct registry/registrar or RDAP service.
- Private/redacted fields: expected privacy protection.
- Stale dates or inconsistent fields: records and display services can differ.

## 7. Viva
**Q: What is WHOIS used for?** Registration and allocation information.
**Q: Does WHOIS resolve a domain to an IP?** No; DNS tools do that.
**Q: Why might owner details be hidden?** Privacy services or registry policies.
**Q: WHOIS vs DNS?** WHOIS/RDAP provides registration metadata; DNS provides records used to locate services.
