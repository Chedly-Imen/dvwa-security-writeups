# DVWA Security Write-ups

Write-ups from a hands-on penetration test on DVWA (Damn Vulnerable Web Application),
covering vulnerabilities across the OWASP Top 10. Conducted using Burp Suite and Kali Linux
as part of an engineering internship in offensive security.

## 1. Command Injection

**Risk:** Critical | **OWASP:** A05:2025 – Injection

**Context:** The "Ping a device" tool takes a user-supplied IP address and runs a system ping command against it.

**Vulnerability:** User input is concatenated directly into the shell command without filtering. The `;` character allows chaining arbitrary commands.

**Exploitation:** After confirming command execution with a reference IP, injected `; whoami` and `; pwd ; ls -la` for reconnaissance, then established a reverse shell via Netcat and a bash payload. From the shell, extracted the application's `config.inc.php`, revealing MySQL credentials — cracked with John the Ripper.

**Impact:** Full compromise chain: arbitrary command execution → reverse shell → database credential theft.

**Recommendation:** Never build shell commands from unfiltered input; use native functions instead of `shell_exec`/`system`; strictly validate input format with a whitelist (e.g., regex for IP addresses).

---

## 2. SQL Injection

**Risk:** Critical | **OWASP:** A05:2025 – Injection

**Context:** A "User ID" field returns the first and last name associated with a submitted ID.

**Vulnerability:** User input is concatenated directly into the SQL query, with no prepared statement or escaping.

**Exploitation:** Confirmed the query structure, then injected a `UNION SELECT` payload to extract usernames and password hashes from the `users` table instead of the expected name fields. Hashes were cracked with John the Ripper.

**Impact:** Full database extraction, including authentication data for all accounts.

**Recommendation:** Always use parameterized queries; never build SQL via string concatenation; apply least privilege to the database account; never store passwords as MD5 — use bcrypt or Argon2.

---

## 3. CSRF (Cross-Site Request Forgery)

**Risk:** High | **OWASP:** A01:2025 – Broken Access Control

**Context:** The password change form processes requests via GET with no anti-CSRF token.

**Vulnerability:** The application never verifies that the password change request came from a deliberate user action.

**Exploitation:** Built a malicious HTML page containing an `<img>` tag pointing to the password-change URL with attacker-chosen values. Any authenticated victim opening the page unknowingly triggers the request with their valid session cookies.

**Impact:** An attacker can change a victim's password simply by getting them to open a crafted page — no session theft required.

**Recommendation:** Implement a unique anti-CSRF token per session/form, verified server-side; handle sensitive actions via POST, never GET; use `SameSite=Strict` or `Lax` on cookies.

---

## Tools used
Burp Suite, Kali Linux, John the Ripper, Netcat

## Disclaimer
All testing was performed in a legal, isolated lab environment (DVWA) as part of a supervised engineering internship. No real systems were targeted.
