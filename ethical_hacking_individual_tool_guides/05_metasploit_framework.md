# Metasploit Framework — Modular Security Testing

## 1. Purpose
Metasploit is a framework containing modules for authorized vulnerability validation and security testing. Module categories include auxiliary, exploit, payload, post, and encoder/evasion-related modules; exact categories and behavior vary by version.

**Memory hook:** Metasploit = **a toolbox of modules**, not one single command.

## 2. Inputs
A module may take a target address (`RHOSTS`), target port (`RPORT`), local callback address (`LHOST`) for some payloads, and module-specific options. Only set these for a lab you control.

## 3. Safe starter workflow
```bash
msfconsole
help
search type:auxiliary scanner
db_status
```
Use `info MODULE_NAME` to inspect a module. For a non-invasive lab service check, choose an appropriate scanner module and read its description/options before running it. Example framework navigation:
```text
search type:auxiliary
use MODULE_PATH
show options
info
```
The exact module path depends on the task and installed version. Do not run an exploit simply to see whether it works.

## 4. Understanding options and output
- `RHOSTS`: remote target(s) the module will contact.
- `RPORT`: remote service port.
- `LHOST`: local callback/listener address, where relevant.
- `show options`: required and optional fields, current values, and descriptions.
- `check` may be supported by some exploit modules; not every module supports a reliable check.
- Output such as “completed” means the module ran; interpret the actual findings rather than equating completion with vulnerability.

## 5. Output explanation
A scanner may report a detected service/version or a finding. Explain what the module tested, the evidence it returned, and any limits. A timeout can mean filtering, routing problems, wrong target, or a service that is unavailable.

## 6. Database
Inside `msfconsole`:
```text
db_status
```
A connected message indicates the framework can use its configured database. It does not mean a scan has been run or findings exist.

## 7. Viva
**Q: What is a module?** A reusable component for a defined task.
**Q: Exploit vs payload?** An exploit attempts to trigger a weakness; a payload is code/action delivered if the exploit succeeds.
**Q: What is `RHOSTS`?** The target host(s).
**Q: Why inspect `info` and `show options`?** To understand module behavior and required inputs before execution.
**Q: Does a successful scan equal a successful exploit?** No.
