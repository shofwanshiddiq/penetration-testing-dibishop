# III. Reconnaissance & Attack Narrative

[← Back to README](./README.md)

---

## 3.1 Public Information (OSINT)

- **Technologies used:** Nmap, Gobuster, Whatweb
- **Open Ports:** 22, 80, 443, 888, 7800

---

## 3.2 Application Mapping

### 3.2.1 Nmap Scan Results

**Command:** `nmap -sV dibishop.duckdns.org`

| **Port** | **Status** | **Service** | **Version** |
|---|---|---|---|
| 22/tcp | **OPEN** | SSH | OpenSSH 8.9p1 Ubuntu |
| 80/tcp | **OPEN** | HTTP | Apache httpd |
| 443/tcp | **OPEN** | HTTPS | Apache httpd |
| 888/tcp | **OPEN** | HTTP | Apache httpd |
| 7800/tcp | **OPEN** | HTTP | Nginx (phpMyAdmin) |
| 20/tcp | **CLOSED** | FTP-data | - |
| 21/tcp | **CLOSED** | FTP | - |

---

### 3.2.2 Whatweb Results

**Command:** `whatweb http://dibishop.duckdns.org`

```
[200 OK] Apache, Bootstrap, Cookies[PHPSESSID], Country[RUSSIAN FEDERATION],
Email[bimbing.id], HTML5, HTTPServer[Apache], IP[188.166.209.84], Script,
Title[Dibimbing-Shop], UncommonHeaders[upgrade], Compatible[IE=edge]
```

**Analysis:**

| **Item** | **Finding** |
|---|---|
| Web Server | Apache |
| Backend Language | PHP (PHPSESSID) |
| Frontend | HTML5 + Bootstrap |
| Session Management | PHPSESSID cookie |
| IP Address | 188.166.209.84 (cloud/VPS hosted) |
| Server Location | *Russian Federation* (possibly VPS/reverse-proxy) |
| Email | bimbing.id |
| Uncommon Headers | `upgrade` (potential misconfiguration indicator) |
| Browser Compatibility | Supports IE=edge |

---

### 3.2.3 Gobuster Directory Enumeration Results

| **Path** | **Status Code** | **Notes** |
|---|---|---|
| `/.git/HEAD` | 200 OK | Git repository is publicly exposed |
| `/admin_area/` | 301 Redirect | Admin panel found and publicly accessible |
| `/customer/` | 301 Redirect | Customer area |
| `/functions/` | 301 Redirect | PHP functions directory |
| `/includes/` | 301 Redirect | PHP includes directory |
| `/index.php` | 200 OK | Main homepage |
