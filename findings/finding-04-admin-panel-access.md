# Finding 4: Admin Panel Access (Broken Access Control)

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 9.8 (Critical)**  
**Category (OWASP):** Broken Access Control  
**Affected Endpoint:** `http://dibishop.duckdns.org/admin_area/login.php`

---

## Risk

This vulnerability enables unauthorized access to the admin panel using credentials obtained through previous vulnerabilities (SQL Injection and Plaintext Password). An attacker can leverage all admin features including user management, customer management, products, orders, pricing, and other admin accounts.

---

## Severity

Classified as **Critical** because it provides full, unrestricted access to highly sensitive administrative functions.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Access performed through the admin endpoint over the internet. |
| Attack Complexity | Low | No additional protection mechanisms such as MFA — only basic login required. |
| Privileges Required | Low | Requires admin login, but credentials are easily obtained via SQL Injection. |
| User Interaction | None | No additional interaction needed. |
| Scope | Unchanged | Impact remains within the application system. |
| Confidentiality Impact | High | Other admin data, customers, products, and orders can be accessed. |
| Integrity Impact | High | All data can be modified. |
| Availability Impact | High | System can be sabotaged or disrupted. |

---

## Source Code Evidence (`admin_area/dashboard.php`)

No token validation, rate limiting, or IP whitelisting exists on the admin login page. Only session validation is present — which is already satisfied since admin credentials were obtained via SQL Injection.

---

## Exploitation Steps

1. Navigate to `http://dibishop.duckdns.org/admin_area/login.php`
2. Enter the admin email and password obtained from the database dump (see [Finding 1](./finding-01-sql-injection.md) and [Finding 3](./finding-03-plaintext-password.md)).
3. Full admin access is granted immediately — no additional barriers exist.

---

## Impact

The attacker gains full control of the admin panel, including:
- Viewing and modifying all admin accounts
- Accessing all customer data (names, emails, addresses, order history)
- Modifying product names and prices
- Manipulating or deleting order records
- Creating fraudulent transactions

---

## Mitigation

- Implement **Multi-Factor Authentication (MFA)** on the admin panel login.
- Perform an immediate **password reset** for all admin users.
- Apply **IP Whitelisting** so the admin dashboard is only accessible from trusted IP addresses.
- Use **session regeneration** (`session_regenerate_id(true)`) after successful login.
- Implement **rate limiting** and **account lockout** after several failed login attempts.
