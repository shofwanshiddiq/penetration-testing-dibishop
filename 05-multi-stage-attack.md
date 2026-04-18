# VI. Multi-Stage Attack

[← Back to README](./README.md)

---

The following summarizes the complete end-to-end attack scenario successfully demonstrated during testing. It illustrates how an attacker can achieve full admin panel access by chaining multiple interconnected vulnerabilities.

| **Stage** | **Attacker Action** | **Vulnerability Exploited** | **Outcome** |
|---|---|---|---|
| 1. Recon | Port scan + gobuster + `curl .git/HEAD` | Security Misconfiguration | Obtained GitHub repository URL → full source code access |
| 2. Analysis | Reviewed PHP source code from GitHub | Git Exposure, Broken Access Control | Identified SQLi vulnerability in `cart.php` |
| 3. SQLi | SQLMap database dump via coupon field | SQL Injection | Obtained 3 admin credentials in plain text |
| 4. Auth Bypass | Logged into admin panel with stolen credentials | Plaintext Password | Attacker obtained email and password to log in as admin |
| 5. XSS *(optional)* | Injected XSS payload into profile name | Stored Cross-Site Scripting | Customer/admin session cookies can be stolen |
| 6. Post-Exploit | Used admin dashboard features illegally: fraudulent transactions, deleting user data, modifying prices, etc. | Admin Panel Access | Data breach, data manipulation, financial losses, etc. |

---

## Attack Chain Visualization

```
[Public Internet]
       │
       ▼
 [1. Recon] ──────────────────────► Source Code Exposed (.git)
       │
       ▼
 [2. Analysis] ─────────────────── White-box review → SQL Injection found
       │
       ▼
 [3. SQL Injection] ─────────────► Full DB dump (admin credentials, customer data)
       │
       ▼
 [4. Auth Bypass] ───────────────► Admin panel access (plain text password)
       │
       ├──────────────────────────► [5. XSS (optional)] → Session hijacking
       │
       ▼
 [6. Post-Exploitation]
   ├── Modify product prices
   ├── Manipulate orders
   ├── Delete customer data
   ├── Create fraudulent transactions
   └── Full system compromise
```

> The entire attack chain was executed in a **single testing session** using only basic Kali Linux tools, without requiring special credentials or network access (e.g., intranet).
