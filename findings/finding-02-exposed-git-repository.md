# Finding 2: Exposed Git Repository

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 7.5 (High)**  
**Category (OWASP):** Broken Access Control  
**Affected Path:** `/.git/`

---

## Risk

The attacker gained access to the application's full source code, including commit history and configuration files. This significantly facilitates in-depth white-box analysis to find other vulnerabilities such as SQL Injection, hardcoded credentials, or API keys.

---

## Severity

Classified as **High** because, although it does not directly grant system access, it dramatically increases the chances of exploiting other vulnerabilities.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Repository accessible via public URL. |
| Attack Complexity | Low | No special technique required — simply access `/.git/`. |
| Privileges Required | None | No authentication needed. |
| User Interaction | None | No user interaction required. |
| Scope | Unchanged | No direct impact on the running application. |
| Confidentiality Impact | High | Source code and internal configuration are exposed. |
| Integrity Impact | None | No direct system changes. |
| Availability Impact | None | Does not directly affect system availability. |

---

## Exploitation Steps & Evidence

1. Access `http://dibishop.duckdns.org/.git/HEAD` → returns: `ref: refs/heads/main`

    <img width="428" height="57" alt="image" src="https://github.com/user-attachments/assets/c6a096be-0e71-40f1-a3be-94ecb40a6a53" />

3. Access `http://dibishop.duckdns.org/.git/config` → returns the remote GitHub repository URL.

    <img width="586" height="147" alt="image" src="https://github.com/user-attachments/assets/08db9495-fdc5-4d28-bd74-1228728ce8a4" />


5. From `.git/logs/HEAD`, confirmed the server runs as `root` on a VPS.

    <img width="857" height="153" alt="image" src="https://github.com/user-attachments/assets/dfaccbe7-cfbb-409b-bbf8-a3cdf28e1e0b" />

7. Attacker accesses the public GitHub repository: [https://github.com/rezafahmialviandy/Dibimbing-Shop](https://github.com/rezafahmialviandy/Dibimbing-Shop)
8. With source code access, the attacker can conduct effective white-box penetration testing.

---

## White-Box Source Code Analysis Results

Vulnerabilities confirmed through source code review:

- SQL Injection in `cart.php`, `details.php`, `customer_register.php`, `checkout.php`
- Price manipulation via DevTools in `details.php`
- Cross-Site Scripting in `details.php`
- Plaintext password storage in `customer_register.php`

---

## Impact

The exposed repository enables in-depth white-box testing, allowing the attacker to map the entire application structure and efficiently identify further exploitable vulnerabilities.

---

## Mitigation

Block access to the `.git` directory via `.htaccess`:

```apache
<Directory ~ "\.git">
    Order allow,deny
    Deny from all
</Directory>
```

Additional steps:
- Remove the `.git` directory from the production web root entirely.
- Ensure the GitHub repository is set to **private** if it contains production code.
