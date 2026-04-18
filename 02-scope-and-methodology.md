# II. Scope & Methodology

[← Back to README](./README.md)

---

## 2.1 Test Target

| **Parameter** | **Details** |
|---|---|
| Target URL | http://dibishop.duckdns.org |
| IP Address | 188.166.209.84 |
| Roles Tested | Admin, User, Unauthorized User (Not Logged In) |
| Server | Apache / Ubuntu Linux |
| Backend | PHP, MySQL |
| Repository | https://github.com/rezafahmialviandy/Dibimbing-Shop |

---

## 2.2 Rules of Engagement (RoE)

Testing was conducted within the following boundaries (as per the RoE document):

**Permitted:**
- Active scanning (Nmap / Burp Suite)
- Manual exploitation
- Proof of Concept (PoC)

**Prohibited:**
- Denial of Service (DoS)
- Permanent data destruction
- Web defacement
- Damage to server OS file system
- All penetration testing activities are documented for educational purposes and completion of the final assignment only.

---

## 2.3 Tools Used

| **Tool** | **Category** | **Description** |
|---|---|---|
| Kali Linux | OS | Platform for running Nmap, Gobuster, Whatweb, Curl, Burp Suite, SQLMap |
| Nmap | Reconnaissance | Open port and service version scanning |
| Gobuster | Reconnaissance | Directory enumeration |
| Whatweb | Reconnaissance | Technology fingerprinting and version detection |
| Curl | Reconnaissance | HTTP header inspection |
| Burp Suite | Vulnerability Analysis | Intercept and analyze HTTP request/response |
| SQLMap | SQL Injection | Automated SQL Injection testing and data dumping |
| DevTools | Testing Scenario | Cookie inspection, XSS scenario testing |
| Git Clone | Vulnerability Analysis | Clone exposed Git repository |
