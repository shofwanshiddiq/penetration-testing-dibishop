# Finding 6: Insecure Cookie

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 6.5 (Medium)**  
**Category (OWASP):** Security Misconfiguration  
**Affected Cookie:** `PHPSESSID`

---

## Risk

Cookies without `HttpOnly` and `Secure` flags can be stolen via XSS or unencrypted connections, enabling session hijacking.

---

## Severity

Classified as **Medium** because it requires additional conditions — such as an active XSS vulnerability — for full exploitation.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Exploitation occurs over the network. |
| Attack Complexity | Low | Only requires the XSS attack (via profile edit) to expose the cookie. |
| Privileges Required | None | No prior login required to exploit XSS as the vector. |
| User Interaction | Required | User must access the page containing the XSS payload. |
| Scope | Unchanged | Does not extend to other systems. |
| Confidentiality Impact | High | Session ID can be stolen and used for impersonation. |
| Integrity Impact | None | Does not directly modify data. |
| Availability Impact | None | Does not directly delete data. |

---

## Evidence

When the XSS script runs (see [Finding 5](./finding-05-stored-xss.md)), the `PHPSESSID` value is directly visible via `document.cookie` and displayed in an alert dialog — confirming the cookie lacks the `HttpOnly` flag.

---

## Attack Chain

The combination of **Stored XSS** (Finding 5) and **Insecure Cookie** (this finding) enables a complete session hijacking attack:

1. Attacker injects XSS payload into the Customer Name field.
2. When any user (or admin) views the page, the script runs and reads `document.cookie`.
3. Because `HttpOnly` is not set, JavaScript can access the `PHPSESSID` value.
4. Because `Secure` is not set, the cookie can also be transmitted over plain HTTP.
5. The stolen session ID is sent to the attacker's server, enabling account takeover.

---

## Mitigation

- Implement **HTTPS** with **TLS 1.3+** across the entire application.
- Set the `HttpOnly` and `Secure` flags on all session cookies:

```php
// ✅ In php.ini (production)
session.cookie_httponly = 1
session.cookie_secure = 1

// ✅ Or at runtime
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
session_start();
```
