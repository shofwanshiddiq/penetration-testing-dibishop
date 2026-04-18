# Finding 8: Session Fixation (+ Missing CSRF Protection)

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 5.8 (Medium)**  
**Category (OWASP):** Insecure Design  
**Affected Pages:** User login, Admin login

---

## Risk

An attacker can force a user to authenticate using a predetermined session ID. Because the session ID is not regenerated after login, the attacker — who already knows the session ID — gains access to the authenticated session without ever obtaining the user's credentials.

---

## Severity

Classified as **Medium** because it requires user interaction and social engineering to execute successfully.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Performed via web request. |
| Attack Complexity | High | Requires trapping the user with a crafted URL — social engineering needed. |
| Privileges Required | None | No account required for the attacker. |
| User Interaction | Required | Victim must click the crafted link and log in. |
| Scope | Changed | If victim is an admin, scope expands to the full admin panel and its features. |
| Confidentiality Impact | Low | Does not directly steal data — requires further attack using the hijacked session. |
| Integrity Impact | Low | Does not directly affect integrity — requires further attack. |
| Availability Impact | None | Does not directly disrupt availability. |

---

## Exploitation Steps

1. Attacker obtains a valid session ID from the application (before any login).
2. Attacker crafts a URL embedding that session ID and delivers it to the victim via social engineering (e.g., phishing email, fake link).
3. Victim clicks the link and logs in normally using their credentials.
4. Because the session ID is **not regenerated** after login, the attacker now holds the same authenticated session.
5. Attacker can now access the application as the victim.

---

## Evidence

**Admin Panel:** Session ID before login = Session ID after login *(no change)*.

**User Panel:** Session ID before login = Session ID after login *(no change)*.

The `session_regenerate_id()` call is absent from both the admin and user login handlers.

Additionally, no **CSRF token** is present on either login form, leaving them further exposed.

---

## Impact

- If the victim is a **regular user**: attacker gains access to their account and order history.
- If the victim is an **admin**: attacker gains full admin panel access, extending the scope significantly.

---

## Mitigation

Add `session_regenerate_id(true)` immediately after a successful login:

```php
// ✅ Fixed — admin login handler
if ($count == 1) {
    session_regenerate_id(true); // Destroy old session ID, generate new one
    $_SESSION['admin_email'] = $admin_email;
}

// ✅ Fixed — user login handler
if ($count == 1) {
    session_regenerate_id(true);
    $_SESSION['customer_id'] = $customer_id;
}
```

Additionally:
- Implement **CSRF tokens** on all forms that perform state-changing actions.
- Set a short **session expiry** to limit the window of opportunity.
