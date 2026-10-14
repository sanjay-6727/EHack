# nslookup — DNS Query Tool

## 1. Purpose
`nslookup` asks a DNS server for records such as A, AAAA, MX, and NS records.

**Memory hook:** DNS is the internet's **phonebook**; nslookup asks for an entry.

## 2. Inputs
- Domain name.
- Optional record type.
- Optional DNS server.

## 3. Commands
```bash
nslookup example.com
nslookup -type=A example.com
nslookup -type=AAAA example.com
nslookup -type=MX example.com
nslookup -type=NS example.com
nslookup example.com 1.1.1.1
```
Syntax can vary slightly by platform; the interactive form also works:
```text
nslookup
set type=MX
example.com
exit
```

## 4. Typical output
```text
Name:    example.com
Address: 93.184.216.34
```
This address is an illustration and can change.

## 5. Explain the output
- **Name:** queried DNS name.
- **Address:** answer returned by the DNS server.
- **MX:** mail exchanger records.
- **NS:** authoritative name server names.
- **Non-authoritative answer:** a resolver supplied a cached/recursive answer rather than acting as the authoritative source.
Multiple addresses may indicate load balancing, CDNs, or multiple endpoints. A missing answer does not always mean the domain is invalid.

## 6. Troubleshooting
- `Non-existent domain` / NXDOMAIN: the queried name was not found in DNS at that time.
- Timeout: server unreachable, filtering, or DNS issue.
- Different servers return different results: caching, split DNS, geography, or recent changes may explain it.

## 7. Viva
**Q: What is DNS?** A distributed naming system that maps names to records.
**Q: A vs AAAA?** IPv4 address vs IPv6 address.
**Q: MX?** Mail exchange record.
**Q: Does DNS output show the physical server location?** No.
