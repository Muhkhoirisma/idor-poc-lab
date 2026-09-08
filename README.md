# 🔐 Security Assessment Report — [Vulnerability Type] on [Lab/Fictional Target Name]

> ⚠️ **Disclaimer:** This report is based on a self-hosted lab environment / fictional application built for educational and portfolio purposes. No real production system, third-party service, or user data was involved. All hostnames, tokens, and identifiers shown below are synthetic.

---

## 📋 Summary

| Field | Detail |
|---|---|
| **Target** | `lab-app.local` (self-hosted / DVWA / OWASP Juice Shop / custom API) |
| **Vulnerability Class** | Broken Access Control — Insecure Direct Object Reference (IDOR) |
| **Severity** | High (CVSS 3.1: 8.1 — customize per case) |
| **Component** | `/api/v1/orders/{order_id}` endpoint |
| **Status** | Demonstrated in lab environment |

---

## 🎯 Objective

To demonstrate how missing object-level authorization checks on a REST API endpoint allow an authenticated user to access resources belonging to another account, using a controlled lab setup rather than a live production target.

---

## 🧪 Environment Setup

- Application: [Juice Shop / DVWA / custom Flask-Django API / etc.]
- Two test accounts created locally: `userA@test.local` and `userB@test.local`
- No real user data, all accounts and records were seeded for testing

---

## 🔍 Methodology

1. **Reconnaissance** — Mapped available API endpoints using Burp Suite / Postman while authenticated as `userA`
2. **Baseline Request** — Captured a normal request as `userA` retrieving their own order:
   ```
   GET /api/v1/orders/1001
   Authorization: Bearer <userA_token>
   ```
3. **Authorization Test** — Repeated the same request while substituting `userB`'s known object ID, still using `userA`'s session token
4. **Observation** — Recorded whether the server validated ownership of the resource before returning data

---

## 📸 Proof of Concept

**Request (as userA, requesting userB's resource):**
```http
GET /api/v1/orders/1002 HTTP/1.1
Host: lab-app.local
Authorization: Bearer eyJhbGciOi...[userA_token_truncated]
```

**Response:**
```json
{
  "order_id": 1002,
  "owner": "userB@test.local",
  "items": ["..."],
  "shipping_address": "..."
}
```

> Expected behavior: server should return `403 Forbidden` since the resource belongs to a different account than the one authenticated.

---

## 💥 Impact

If this pattern existed in a production system, it could allow:
- Unauthorized access to other users' personal or transactional data
- Potential horizontal privilege escalation across accounts
- Data enumeration by iterating sequential object IDs

---

## 🛠 Root Cause

The endpoint retrieves the object by ID directly from the URL path without verifying that `object.owner_id == authenticated_user.id` before returning the response.

---

## ✅ Remediation

1. Enforce object-level authorization checks server-side for every request that accesses a resource by ID
2. Use indirect reference maps (e.g., UUIDs scoped to the session) instead of predictable sequential IDs
3. Add automated authorization test cases (e.g., "user A cannot access user B's resource") to the CI/CD pipeline
4. Implement centralized access-control middleware rather than per-endpoint checks

---

## 🧾 Notes

This report intentionally uses a lab/self-hosted environment so that full technical detail — including request/response pairs — can be shared publicly without violating any responsible disclosure agreement. When documenting real bug bounty findings, omit exact payloads, tokens, and target identifiers, and follow the program's disclosure policy before publishing anything.

---

## 👤 Author

Linkedin : https://www.linkedin.com/in/muh-khoirisma-65478742b/
HackerOne: https://www.hackerone.com/attack10
CyberArmy : https://app.cyberarmy.id/bughunter/profile
vercel : https://muh-khoirisma.vercel.app/
