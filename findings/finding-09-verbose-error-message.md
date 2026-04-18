# Finding 9: Verbose SQL Error Message

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 5.3 (Medium)**  
**Category (OWASP):** Security Misconfiguration  
**Affected Field:** Coupon code input on `cart.php`

---

## Risk

Error messages expose internal application details — including database query structure and server file paths — which assist an attacker in crafting and refining SQL Injection attacks.

---

## Severity

Classified as **Medium** because it has no direct standalone impact, but actively supports other attacks such as SQL Injection (see [Finding 1](./finding-01-sql-injection.md)).

---

## CVSS Base Score Breakdown

| **Metric** | **Value** |
|---|---|
| Attack Vector | Network |
| Attack Complexity | Low |
| Privileges Required | None |
| User Interaction | None |
| Scope | Unchanged |
| Confidentiality Impact | Low |
| Integrity Impact | None |
| Availability Impact | None |

---

## Evidence

When a special SQL character (e.g., `'`) is entered in the coupon code field, the application returns a detailed MySQL error message in the browser — revealing:

<img width="1132" height="141" alt="image" src="https://github.com/user-attachments/assets/4159c104-b2cd-4ebd-9cd8-a72600059539" />


- The raw SQL query structure being executed
- The server-side file path (e.g., `/var/www/html/cart.php`)
- The line number where the error occurred

This information directly accelerated the SQL Injection exploitation in Finding 1.

---

## Impact

- Provides attackers with internal query structure, making SQL Injection easier to craft.
- Reveals server file paths useful for directory traversal or targeted file inclusion attacks.
- Reduces attacker effort significantly — effectively acting as an information oracle.

---

## Mitigation

- Disable `display_errors` in production:

```ini
; php.ini (production)
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log
```

- Use try-catch blocks to handle database exceptions gracefully:

```php
// ✅ Fixed
try {
    $result = $con->query($sql);
} catch (Exception $e) {
    // Log internally, show generic message to user
    error_log($e->getMessage());
    echo "An error occurred. Please try again.";
}
```

- Display only generic error messages to end users — never expose query details, file paths, or stack traces.
