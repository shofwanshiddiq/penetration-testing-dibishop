# VII. Risk Assessment

[← Back to README](./README.md)

---

Risk assessment was conducted based on a combination of **Likelihood** (probability of exploitation) and **Impact** (effect if successfully exploited), with reference to the **OWASP Risk Rating Methodology**.

| **No** | **Finding** | **CVSS** | **Likelihood** | **Impact** | **Risk Level** |
|---|---|---|---|---|---|
| 1 | SQL Injection on coupon code input in Cart | High (7.5) | High | High | **High** |
| 2 | Exposed Git Repository | High (7.5) | High | High | **High** |
| 3 | Password stored in plaintext | High (8.1) | High | High | **High** |
| 4 | Admin account exposed | Critical (9.8) | High | Very High | **Critical** |
| 5 | Stored Cross-Site Scripting | Critical (9.3) | Medium | Very High | **High** |
| 6 | Insecure Cookie | Medium (6.5) | Medium | Medium | **Medium** |
| 7 | Unrestricted file upload | Medium (6.5) | High | Low | **Low** |
| 8 | Session Fixation (CSRF) | Medium (5.8) | Low | Medium | **Medium** |
| 9 | Verbose SQL Error Message | Medium (5.3) | High | Low | **Low** |

---

## Risk Matrix

```
          │  Low Impact  │ Medium Impact │ High Impact │ Very High Impact
──────────┼──────────────┼───────────────┼─────────────┼──────────────────
High      │  Low         │  Medium       │  High       │  Critical
Likelihood│  (#7, #9)    │               │  (#1,#2,#3) │  (#4)
──────────┼──────────────┼───────────────┼─────────────┼──────────────────
Medium    │              │  Medium       │  High       │
Likelihood│              │  (#6)         │  (#5)       │
──────────┼──────────────┼───────────────┼─────────────┼──────────────────
Low       │              │  Medium       │             │
Likelihood│              │  (#8)         │             │
```
