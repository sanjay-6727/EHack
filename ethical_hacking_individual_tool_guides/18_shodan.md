# Shodan — Search Engine for Internet-Connected Devices

## 1. Purpose
Shodan indexes observations about internet-reachable devices and services. It is not the same as actively scanning a target from your own machine, and its records may be stale.

**Memory hook:** Shodan = **a catalogue of what the internet has observed**.

## 2. Inputs
- Search filters/terms.
- Optional account for additional query features.
- A domain, organization, port, product, or network that you are authorized to research.

## 3. Browser workflow
1. Open `https://www.shodan.io/`.
2. Enter a narrow query.
3. Inspect the result's observation date, IP, port, service/banner, and context.
4. Treat the result as a clue and validate only assets in scope.

Example query patterns:
```text
hostname:example.com
port:443
country:IN
org:"Example Organization"
```
Use the site's current search syntax and ensure you are not querying private or unauthorized assets for active testing.

## 4. Output and interpretation
- **IP/host:** address associated with the observation.
- **Port:** externally observed service port.
- **Banner/product:** service response or fingerprint.
- **Last seen/observation date:** helps judge freshness.
- **Organization/location fields:** estimates from data sources, not guaranteed physical location.

Explain: “Shodan shows that a service was observed at this address and port at a particular time. I would verify current exposure and ownership before drawing conclusions.”

## 5. Limitations
- Data can be stale.
- A banner may be misleading or incomplete.
- Internet exposure is not itself proof of a vulnerability.
- Do not use search results as permission to probe or exploit.

## 6. Viva
**Q: Shodan vs Nmap?** Shodan searches indexed internet observations; Nmap actively sends probes from the scanner.
**Q: What is a banner?** Text or protocol data returned by a service that can help identify it.
**Q: Does Shodan show every device?** No.
