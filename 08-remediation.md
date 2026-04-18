# VIII. Remediation Steps

[← Back to README](./README.md)

---

| **Priority** | **Vulnerability** | **Remediation Action** | **Timeline** |
|---|---|---|---|
| 1 | SQL Injection & Plaintext Password | Implement Prepared Statements; hash passwords with bcrypt/Argon2 | < 1 day |
| 2 | Exposed Git Repository | Block `.git/` access via `.htaccess`; remove from web root | < 1 day |
| 3 | No File Upload Validation | Add file type validation (whitelist: jpg, jpeg, png, gif); rename uploaded files randomly | < 2–3 days |
| 3 | Exposed Git Repository | Ensure production GitHub repository is set to private | < 1 day |
| 4 | Stored Cross-Site Scripting | Implement `htmlspecialchars()` and Content Security Policy (CSP) headers | < 1 week |
| 5 | Admin Panel Access | Add MFA, rate limiting, and IP whitelisting for admin login | < 1 week |
| 6 | Insecure Cookie & Session Fixation | Set cookie flags (`HttpOnly`, `Secure`); regenerate Session ID after login | < 1 month |
| 7 | Verbose Error Message | Implement generic error handling in production; disable `display_errors` | < 1 month |

---

## Quick Reference — Code Fixes

### SQL Injection → Prepared Statements

```php
// ❌ Vulnerable
$query = "SELECT * FROM coupons WHERE coupon_code='$code'";

// ✅ Fixed
$stmt = $con->prepare("SELECT * FROM coupons WHERE coupon_code = ?");
$stmt->bind_param("s", $code);
$stmt->execute();
```

### Plaintext Password → Hashing

```php
// ❌ Vulnerable
$password = $_POST['password']; // stored as-is

// ✅ Fixed
$password = password_hash($_POST['password'], PASSWORD_BCRYPT);
```

### Block .git Access (`.htaccess`)

```apache
<Directory ~ "\.git">
    Order allow,deny
    Deny from all
</Directory>
```

### File Upload Validation

```php
$allowed = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($c_image, PATHINFO_EXTENSION));
if (!in_array($ext, $allowed)) {
    die('File type not allowed');
}
```

### XSS → Output Encoding

```php
echo htmlspecialchars($customer_name, ENT_QUOTES, 'UTF-8');
```

### Session Fixation → Regenerate Session ID

```php
if ($count == 1) {
    session_regenerate_id(true);
    $_SESSION['admin_email'] = $admin_email;
}
```

### Insecure Cookie → Set Secure Flags

```php
// In php.ini or at runtime:
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
```

### Verbose Error Message → Generic Errors

```php
// php.ini (production)
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log
```
