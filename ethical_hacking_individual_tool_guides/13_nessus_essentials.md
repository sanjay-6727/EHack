# Nessus Essentials — Vulnerability Assessment

## 1. Purpose
Nessus scans systems for known vulnerabilities, misconfigurations, and exposed services using plugins and checks. A finding is an assessment result that may require manual validation.

**Memory hook:** Nessus = **the building inspector**.

## 2. Inputs
- Authorized target IP/hostname.
- Scan type/template.
- Optional credentials for a credentialed assessment.
- Scan policy settings and network reachability.
Only scan your own lab or approved assets.

## 3. GUI workflow
1. Open the Nessus web interface locally (the URL/port is shown by your installation).
2. Sign in with the account tied to your licensed/registered edition.
3. Create a scan, commonly a basic network scan.
4. Enter the Metasploitable 2 lab IP as the target.
5. Review scan settings and launch.
6. Open results and inspect each finding's severity, evidence, affected host/port, and remediation.

## 4. Output and interpretation
- **Critical/High/Medium/Low/Info:** severity categories based on the scanner's assessment; exact scoring depends on plugin and configuration.
- **Plugin name/ID:** the check that produced the result.
- **Port/service:** where the observation was associated.
- **Evidence/output:** data supporting the finding.
- **Remediation:** recommended fix or mitigation.

Explain: “The scanner reported a finding because plugin X observed evidence Y on host/port Z. I would validate the evidence and account for false positives before concluding the system is vulnerable.”

## 5. False positives and limitations
- A version string can be misleading or backported.
- Some checks are unauthenticated and cannot inspect internal patch state.
- Scan results depend on plugins, credentials, reachability, and scan policy.
- A clean scan does not prove a system is perfectly secure.

## 6. Account switching note
If switching Nessus accounts/editions, use the product's supported sign-out/activation workflow and confirm the correct edition/license in the UI. Do not assume that changing a browser account changes the scanner's activation state.

## 7. Viva
**Q: Vulnerability vs risk?** A vulnerability is a weakness; risk combines likelihood and impact in context.
**Q: What is a false positive?** A finding that is reported but not actually applicable/valid.
**Q: Credentialed vs uncredentialed scan?** Credentialed scans can inspect more internal configuration when authorized credentials are supplied.
**Q: Does a high severity score guarantee exploitation?** No; context and validation matter.
