# SQL Injection — Login Bypass

**Platform:** PortSwigger Web Security Academy
**Lab URL:** https://portswigger.net/web-security/sql-injection/lab-login-bypass
**Difficulty:** Apprentice
**Date Completed:** June 2025
**Status:** ✅ Solved

---

## Objective

Log in to the application as the `administrator` user by exploiting a SQL injection
vulnerability in the login function — without knowing the password.

---

## Vulnerability

**Type:** SQL Injection — Authentication Bypass
**Location:** Username field in the login form
**Root Cause:** User input is directly concatenated into a SQL query without sanitization
or parameterized queries.

The backend login query likely looks like this:

```sql
SELECT * FROM users WHERE username = 'input' AND password = 'input'
```

If this query returns any row, the user is authenticated.

---

## Exploitation

### Payload used

```
username: administrator'--
password: (anything / left blank)
```

### Why this works

The single quote `'` closes the string literal in the SQL query early.
The double dash `--` is a SQL comment — it causes the database to ignore
everything that follows, including the `AND password = '...'` check.

The query becomes:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

Which the database processes as:

```sql
SELECT * FROM users WHERE username = 'administrator'
```

The password check is completely removed. Since a row for `administrator` exists,
the application logs in the attacker as that user.

---

## Steps Taken

1. Opened the lab and navigated to the login page.
2. Intercepted the login request using Burp Suite (Proxy > Intercept).
3. Modified the `username` parameter to: `administrator'--`
4. Forwarded the request.
5. Application authenticated and logged in as `administrator`.
6. Lab marked as solved.

---

## Impact

An attacker with no valid credentials can gain full access to any account —
including administrator accounts — by manipulating the login query.
In a real application this would mean complete account takeover.

**CVSS Severity:** High (authentication control fully bypassed)

---

## Remediation

| Fix | Description |
|-----|-------------|
| Parameterized queries | Use prepared statements so user input is never interpreted as SQL |
| Input validation | Reject or sanitize special characters like `'` and `--` |
| Least privilege | Database account used by the app should not have admin-level access |
| WAF | A Web Application Firewall can detect and block common SQLi patterns |

---

## References

- PortSwigger: https://portswigger.net/web-security/sql-injection
- OWASP Top 10 - A03:2021 Injection: https://owasp.org/Top10/A03_2021-Injection/
- CWE-89: Improper Neutralization of Special Elements used in an SQL Command
