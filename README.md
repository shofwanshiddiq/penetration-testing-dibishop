# Penetration Testing and Findings Report

**CONFIDENTIAL**

> **Author:** Muhammad Shofwan Shiddiq  
> **Program:** Bootcamp Cyber Security [Batch 2]  
> **Date:** April 2026  
> **Target:** [http://dibishop.duckdns.org](http://dibishop.duckdns.org)

---

## Table of Contents

1. [Executive Summary](#executive-summary) *(this file)*
2. [Scope & Methodology](./02-scope-and-methodology.md)
3. [Reconnaissance & Attack Narrative](./03-reconnaissance-and-attack-narrative.md)
4. [Finding Summary](./04-finding-summary.md)
5. [Detail Findings & Exploitation](./05-detail-findings.md)
6. [Multi-Stage Attack](./06-multi-stage-attack.md)
7. [Risk Assessment](./07-risk-assessment.md)
8. [Remediation Steps](./08-remediation.md)
9. [Conclusion & References](./09-conclusion-and-references.md)

---

## Executive Summary

### 1.1 Background

Penetration testing was conducted against the **Dibimbing Shop** web application ([http://dibishop.duckdns.org](http://dibishop.duckdns.org)), covering both the user-facing side and the admin panel. A total of **9 vulnerabilities** were identified with the following details.

### 1.2 Finding Summary

| **Vulnerability Severity** | **Count** | **Type of Finding** |
|---|---|---|
| **CRITICAL** | 2 | Stored Cross-Site Scripting, Admin Panel Access |
| **HIGH** | 3 | SQL Injection, Git Repository Leak, Plaintext Password |
| **MEDIUM** | 4 | Session Fixation/CSRF, Verbose Error Message, Unrestricted File Upload, Insecure Cookie |

The most critical finding was the **leaked admin credentials**, which allowed the pentester to log in as an administrator and gain access to all admin menus and features.

### 1.3 Critical Finding Scenario

The following critical attack scenario was successfully demonstrated by the pentester:

1. **Reconnaissance** revealed a security misconfiguration — the `.git` directory was publicly exposed, leading to full access of the Dibimbing-Shop application repository.
2. **SQL Injection** vulnerabilities were found on multiple pages (e.g., `cart.php`), allowing the pentester to dump the database.
3. The **admin table dump** revealed that passwords were stored in **plain text**.
4. The pentester accessed the **admin panel via forced browsing** and logged in using the dumped email and password.
5. The pentester successfully gained full access to all admin panel features and menus.

These varied vulnerabilities, chained into a structured **multi-stage attack**, demonstrate how dangerous seemingly isolated weaknesses can be when combined.

### 1.4 Business Risk

The negative business impact from the identified vulnerabilities includes:

- **Sensitive data breach** — resulting in violations of customer privacy.
- **Access abuse** — enabling fraudulent transactions (price manipulation) and manipulation of sales reports, causing financial losses.
- **Reputational damage** and loss of customer trust.
- **Legal sanctions** (under Indonesia's Personal Data Protection Law / UU PDP) for failing to properly safeguard customer data.
- **Intellectual property loss**.

### 1.5 Recommendations

The primary recommendations are:

- Implement **parameterized queries / prepared statements** across all database inputs.
- Implement **password hashing** (bcrypt or Argon2) — never store passwords in plain text.
- **Restrict public access** to sensitive directories such as `.git` and the admin login page.
- Enforce **HTTPS with TLS 1.3+** across the entire application.
- Add **MFA, rate limiting, and IP whitelisting** on the admin panel.
