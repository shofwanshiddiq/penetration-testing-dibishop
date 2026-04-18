# Finding 3: Password Stored in Plain Text

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 8.1 (High)**  
**Category (OWASP):** Cryptographic Failures  
**Affected Tables:** `admins`, `customers`

---

## Risk

Passwords stored without hashing can be used directly by an attacker after gaining database access. The attacker obtained admin usernames and passwords in fully readable form, enabling login to the admin panel and further exploitation.

---

## Severity

Classified as **High** because, combined with the SQL Injection vulnerability, it forms a multi-stage attack chain that delivers admin credentials directly to the attacker.

> **Note:** If the application did not have an SQL Injection vulnerability, plaintext passwords would not be exposed and the severity would potentially be lower.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Access through the system in the event of a breach. |
| Attack Complexity | High | Requires exploiting SQL Injection with Kali Linux and SQLMap. |
| Privileges Required | None | No admin or regular user account required. |
| User Interaction | None | No user interaction needed. |
| Scope | Unchanged | Impact limited to the same system and database. |
| Confidentiality Impact | High | All passwords are fully exposed. |
| Integrity Impact | High | Exposed admin account enables data modification via the admin panel. |
| Availability Impact | High | Exposed admin account can be used to sabotage data availability. |

---

## Evidence

SQLMap dump of the `admins` table revealed `admin_pass` in fully readable plain text:

| **admin_id** | **admin_name** | **admin_email** | **admin_pass (PLAIN TEXT)** |
|---|---|---|---|
| 12 | fahmi@cakrawala.ac.id | fahmi@cakrawala.ac.id | admindibimbing |
| 15 | Rahasia | rahasia27@gmail.com | rahasia27 |
| 18 | dibi | dibi@shop.org | balonkuadaempat |

---

## Impact

All admin passwords can be used immediately after database exfiltration, giving the attacker full access to the admin panel for further exploitation without any additional cracking step.

---

## Mitigation

- **Never store passwords in plain text, MD5, or SHA1.**
- Use PHP's `password_hash()` with a strong algorithm:

```php
// ✅ Fixed — store hashed password
$hashed = password_hash($_POST['password'], PASSWORD_BCRYPT);

// ✅ Verify on login
if (password_verify($_POST['password'], $hashed_from_db)) {
    // login success
}
```

- Enforce strong password policies (uppercase, numbers, special characters).
- Implement password cycling and periodic rotation.

**Immediate Remediation:**
- Perform a **password reset** for all admin users immediately.
