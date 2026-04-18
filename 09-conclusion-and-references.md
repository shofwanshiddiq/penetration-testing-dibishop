# IX. Conclusion & X. References

[← Back to README](./README.md)

---

## IX. Conclusion

Penetration testing of the Dibimbing-Shop application revealed a **critically insecure security posture**. The most dangerous attack chain identified was:

> **SQL Injection → Database Dump → Plaintext Password → Admin Access → Full System Compromise**

All stages of this chain were successfully executed within a **single testing session**, using only standard Kali Linux tools — without requiring special credentials, a restricted network (intranet), or advanced penetration testing skills.

The root cause across all vulnerabilities is a **lack of input validation and output encoding**, combined with a failure to apply fundamental security configuration principles.

### Key Recommendations

The following measures must be implemented immediately:

1. **Replace all raw SQL queries** with Prepared Statements or Parameterized Queries throughout the application.
2. **Implement password hashing** using bcrypt or Argon2 for both admin and user account tables.
3. **Close public access** to sensitive directories — specifically the `.git` directory and the admin panel login page.
4. **Migrate from HTTP to HTTPS** using a secure TLS configuration (TLS 1.3+).

Implementing these four measures alone would significantly dismantle the attack chain demonstrated during this assessment and substantially reduce the overall risk profile of the application.

---

## X. References

- **CVSS v3.1 Calculator:** [https://www.first.org/cvss/calculator/3.1](https://www.first.org/cvss/calculator/3.1)
- **OWASP Top 10 (2025):** [https://owasp.org/Top10/2025/](https://owasp.org/Top10/2025/)
- **OWASP Risk Rating Methodology:** [https://owasp.org/www-community/OWASP_Risk_Rating_Methodology](https://owasp.org/www-community/OWASP_Risk_Rating_Methodology)
- **Target Application Repository:** [https://github.com/rezafahmialviandy/Dibimbing-Shop](https://github.com/rezafahmialviandy/Dibimbing-Shop)
