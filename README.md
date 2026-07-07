# CVE-2025-70796 — Unauthenticated Path Traversal in WTI Web Management Interface (Camera Control)

**CVE ID:** CVE-2025-70796
**Vulnerability Type:** Directory Traversal / Local File Inclusion (LFI)
**Reported by:** LPO26

---

## Summary

An unauthenticated path traversal vulnerability exists in the web management
interface of WTI (Wireless Technology, Inc.), Camera Control version 3.5.0.r
(build 2024/05/24). An unauthenticated attacker can craft malicious HTTP
requests containing traversal sequences (`../`) to access files outside the
intended web root directory. This allows disclosure of sensitive system files
and configuration data, which may in turn facilitate escalation of privileges
using credentials or configuration recovered from the device.

---

## Affected Product

| Field | Value |
|-------|-------|
| Vendor | WTI (Wireless Technology, Inc.), USA |
| Product | WTI Web Management Interface |
| Component | Camera Control Interface |
| Affected Version | Camera Control 3.5.0.r (build 2024/05/24) |

---

## Vulnerability Details

| Field | Value |
|-------|-------|
| Type | Directory Traversal / Local File Inclusion (LFI) |
| CWE | [CWE-22](https://cwe.mitre.org/data/definitions/22.html) — Improper Limitation of a Pathname to a Restricted Directory |
| Authentication | None (unauthenticated) |
| Attack Type | Remote |
| Attack Vector | Network (HTTP) |
| Impact | Information Disclosure; Escalation of Privileges (follow-on) |

**CVSS 3.1:**
`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` — **7.5 (High)**

*Rationale: network-reachable, low complexity, no privileges or user
interaction required; high confidentiality impact (arbitrary file read,
including credentials and configuration). Integrity and availability are not
directly affected, so the base vector remains confidentiality-only — the
privilege-escalation impact is a downstream consequence of the disclosed data.*

---

## Proof of Concept

The Camera Control endpoint fails to normalize or restrict the requested path,
so traversal sequences resolve against the underlying filesystem. `--path-as-is`
prevents curl from collapsing the `../` sequences client-side, ensuring they
reach the server verbatim. No authentication is required.

```bash
curl "http://TARGET_IP:55003/../../../../../../../../etc/passwd" \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:144.0) Gecko/20100101 Firefox/144.0' \
  --path-as-is
```

Replace `TARGET_IP` with the address of the affected device. A successful
request returns the contents of `/etc/passwd`, confirming the interface serves
files from outside its intended root directory. The same technique can be
pointed at configuration files or credential stores on the device.

---

## Impact

An unauthenticated, remote attacker could:

- Read sensitive system files (e.g., `/etc/passwd`, service configuration)
- Access configuration data and stored credentials
- Use recovered credentials/configuration to escalate privileges or move
  laterally into the connected environment

The net effect is information disclosure with a realistic path to privilege
escalation.

---

## Mitigation

- Apply vendor-provided firmware/software updates once available.
- Restrict access to the web management interface using network controls
  (management VLAN, ACLs, VPN-only access).
- Enforce strong authentication and limit administrative access.
- Disable the web interface if it is not required.
- Monitor device logs for suspicious file-access activity and traversal
  patterns (`../` sequences in request paths).

---

## Disclosure Timeline

| Date | Event |
|------|-------|
| YYYY-MM-DD | Vulnerability discovered |
| YYYY-MM-DD | CVE-2025-70796 assigned by MITRE |
| YYYY-MM-DD | Vendor notified |
| YYYY-MM-DD | Vendor response / patch (if any) |
| YYYY-MM-DD | Public disclosure |

*Fill in the dates as your coordination progresses.*

---

## References

- CWE-22: https://cwe.mitre.org/data/definitions/22.html
- Repository / write-up: https://github.com/LPO26/Path-Traversal-via-Web-Interface-on-WTI-Devices
- Vendor advisory: *(add once published)*
