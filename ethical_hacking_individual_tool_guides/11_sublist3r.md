# Sublist3r — Subdomain Enumeration

## 1. Purpose
Sublist3r attempts to find subdomains using supported public search engines and sources. It is an older tool and some integrations can fail because providers change pages, block requests, or require APIs.

**Memory hook:** Sublist3r = **find names under a domain**.

## 2. Inputs
- A domain in scope, e.g. `example.com`.
- Optional source/provider flags.
- Python and required dependencies.

## 3. Commands
```bash
python sublist3r.py -h
python sublist3r.py -d example.com
```
Some versions support brute-force or additional source flags; use only options shown by your installed `-h` output and only for authorized domains.

## 4. Output and explanation
Typical output is a list of names such as `www.example.com` or `mail.example.com`. Treat these as candidates: resolve them with DNS and verify they belong to the authorized scope.

## 5. Known issue: VirusTotal traceback
If the traceback ends near code like `csrf_regex.findall(resp)[0]` with `IndexError: list index out of range`, the tool expected a token in the response but found none. This often indicates a changed or blocked response, not that the target domain is broken. Missing `termcolor`/`colorama` usually affects colored display, not the core enumeration logic.

Optional display dependencies, if the project documentation calls for them:
```bash
python -m pip install termcolor colorama
```
Prefer a maintained alternative if the old tool no longer works reliably; don't attempt to bypass a provider's blocking.

## 6. Viva
**Q: What is a subdomain?** A name below a parent domain, such as `api.example.com`.
**Q: Does enumeration prove the host is live?** No; verify DNS and service reachability separately.
**Q: Why can results be incomplete?** Search indexes, source limits, DNS changes, and provider restrictions.
**Q: Passive vs brute-force enumeration?** Passive uses existing public information; brute force guesses names and sends additional queries.
