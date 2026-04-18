# V. Finding Summary

[← Back to README](./README.md)

---

## Severity Ratings

The table below defines the severity levels and corresponding **CVSS V3.1** score ranges used throughout this document.

| **Severity** | **CVSS V3.1 Score Range** | **Definition** |
|---|---|---|
| Critical | 9.0 – 10.0 | Exploitation is straightforward and usually results in system-level compromise. Patch immediately. |
| High | 7.0 – 8.9 | Exploitation is more difficult but could cause elevated privileges and data loss. Patch as soon as possible. |
| Moderate | 4.0 – 6.9 | Vulnerabilities exist but require extra steps to exploit. Patch after high-priority issues. |
| Low | 0.1 – 3.9 | Non-exploitable but reduce attack surface. Patch during next maintenance window. |
| Informational | N/A | No vulnerability. Additional information about items noticed during testing. |

### Risk Assessment Factors

- **Likelihood** — the potential for a vulnerability to be exploited, rated on attack difficulty, available tools, attacker skill level, and environment.
- **Impact** — the potential effect on confidentiality, integrity, and availability of systems and data, including reputational and financial damage.

---

## Vulnerability Summary

**Internal Penetration Test Findings**

| Critical | High | Medium |
|---|---|---|
| 2 | 3 | 4 |

| **No** | **Finding** | **Severity** | **CVSS** | **Category (OWASP)** |
|---|---|---|---|---|
| 1 | [SQL Injection on coupon code input in Cart](./findings/finding-01-sql-injection.md) | High | 7.5 | SQL Injection |
| 2 | [Exposed Git Repository](./findings/finding-02-exposed-git-repository.md) | High | 7.5 | Broken Access Control |
| 3 | [Password stored in plaintext](./findings/finding-03-plaintext-password.md) | High | 8.1 | Cryptographic Failures |
| 4 | [Admin account exposed](./findings/finding-04-admin-panel-access.md) | Critical | 9.8 | Broken Access Control |
| 5 | [Stored Cross-Site Scripting](./findings/finding-05-stored-xss.md) | Critical | 9.3 | Cross-Site Scripting |
| 6 | [Insecure Cookie](./findings/finding-06-insecure-cookie.md) | Medium | 6.5 | Security Misconfiguration |
| 7 | [Unrestricted File Upload](./findings/finding-07-unrestricted-file-upload.md) | Medium | 6.5 | Insecure Design |
| 8 | [Session Fixation](./findings/finding-08-session-fixation.md) | Medium | 5.8 | Insecure Design |
| 9 | [Verbose SQL Error Message](./findings/finding-09-verbose-error-message.md) | Medium | 5.3 | Security Misconfiguration |

---

## CVSS Score Calculation

CVSS scores were calculated using the **CVSS Score Calculator v3.1** at:  
[https://www.first.org/cvss/calculator/3.1](https://www.first.org/cvss/calculator/3.1)
