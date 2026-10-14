# Robtex — DNS and IP Intelligence

## 1. Purpose
Robtex is a web-based DNS/IP intelligence service that presents relationships and records associated with domains, IP addresses, and network infrastructure. Results can change and may be cached.

**Memory hook:** Robtex = **a cross-reference desk for internet names and addresses**.

## 2. Inputs
- Domain name, hostname, or IP address.
- Browser and internet access.

## 3. GUI workflow
1. Open `https://www.robtex.com/`.
2. Search for a domain or IP.
3. Review DNS records, related names, and network/ASN information shown.
4. Note the query time and verify important records with a second source such as `nslookup`.

## 4. Output and interpretation
Depending on the query, results may include A/AAAA records, name servers, mail records, reverse DNS names, related IPs, and ASN/network ownership.

Explain: “Robtex provides aggregated DNS and IP context. Its related-name results are investigative leads; they do not necessarily prove that every listed domain shares the same owner.”

## 5. Common issues
- Different result than `nslookup`: caching, geography, record changes, or data-source timing may explain it.
- No data: service may not have indexed the record.
- Related domains: confirm using authoritative records and organizational context.

## 6. Viva
**Q: What is reverse DNS?** Looking up a name associated with an IP, usually via PTR records.
**Q: What is an ASN?** Autonomous System Number, identifying a routing domain under one administrative policy.
**Q: Why verify results?** Aggregated sources may be stale or infer relationships.
