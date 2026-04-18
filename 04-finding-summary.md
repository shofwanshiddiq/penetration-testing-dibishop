# IV. Finding Summary

[← Back to README](./README.md)

---

## 4.1 Severity Ratings

The table below defines the severity levels and corresponding **CVSS V3.1** score ranges used throughout this document to assess vulnerabilities and risk impact.

| **Severity** | **CVSS V3.1 Score Range** | **Definition** |
|---|---|---|
| Critical | 9.0 – 10.0 | Exploitation is straightforward and usually results in system-level compromise. It is advised to form a plan of action and patch immediately. |
| High | 7.0 – 8.9 | Exploitation is more difficult but could cause elevated privileges and potentially a loss of data or downtime. It is advised to form a plan of action and patch as soon as possible. |
| Moderate | 4.0 – 6.9 | Vulnerabilities exist but are not exploitable or require extra steps such as social engineering. It is advised to form a plan of action and patch after high-priority issues have been resolved. |
| Low | 0.1 – 3.9 | Vulnerabilities are non-exploitable but would reduce an organization's attack surface. It is advised to form a plan of action and patch during the next maintenance window. |
| Informational | N/A | No vulnerability exists. Additional information is provided regarding items noticed during testing, strong controls, and additional documentation. |

### Risk Assessment Factors

Risk is measured based on two factors: **Likelihood** and **Impact**.

- **Likelihood** measures the potential for a vulnerability to be exploited, rated based on attack difficulty, available tools, attacker skill level, and the client's environment.
- **Impact** measures the potential effect of a vulnerability on operations, including the confidentiality, integrity, and availability of the client's systems and/or data, reputational damage, and financial loss.

---

## 4.2 Vulnerability Summary & Report Card

The table below illustrates the vulnerabilities found, categorized by **OWASP TOP 10 (2025)** and CVSS score.

**Internal Penetration Test Findings**

| Critical | High | Medium |
|---|---|---|
| 2 | 3 | 4 |

| **No** | **Finding** | **Severity** | **Category (OWASP)** |
|---|---|---|---|
| 1 | SQL Injection on coupon code input in Cart menu | High (7.5) | SQL Injection |
| 2 | Exposed Git Repository | High (7.5) | Broken Access Control |
| 3 | Password stored in plaintext | High (8.1) | Cryptographic Failures |
| 4 | Admin account exposed | Critical (9.8) | Broken Access Control |
| 5 | Stored Cross-Site Scripting | Critical (9.3) | Cross-Site Scripting |
| 6 | Insecure Cookie | Medium (6.5) | Security Misconfiguration |
| 7 | No file upload validation | Medium (6.5) | Insecure Design |
| 8 | Session Fixation | Medium (5.8) | Insecure Design |
| 9 | Verbose SQL Error Message | Medium (5.3) | Security Misconfiguration |

---

## 4.3 CVSS Score Calculation

CVSS scores were calculated using the **CVSS Score Calculator v3.1** at:  
[https://www.first.org/cvss/calculator/3.1](https://www.first.org/cvss/calculator/3.1)
