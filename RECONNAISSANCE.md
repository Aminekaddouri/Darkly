# Darkly v2 — Reconnaissance Notes

> These are the raw notes from the initial reconnaissance phase.
> For the full audit narrative, see [walkthrough.md](walkthrough.md).
> For individual vulnerability details, see each folder under [exploits/](exploits/).

---

## Target

| Property | Value |
|----------|-------|
| URL | http://localhost:4942 |
| Backend | http://localhost:8090 (PocketBase) |
| Version | 1.3.7-prod / Campus: wilcity |
| Stack | FastAPI 0.104 / Python 3.11 / PocketBase 0.22.4 / uvicorn 0.24.0 |
| JWT Secret | "42network" (leaked via XXE) |

---

## Attack Surface

### robots.txt (disallowed paths)

/admin
/staff
/internal
/backup
/api/grades
/_/
/static/uploads/


### Additional endpoints discovered
- `/projects` — project listing + file download
- `/forum` — student forum
- `/newsletter` — email subscription
- `/login` — authentication
- `/reset-password` — password reset flow
- `/api/docs-internal` — API schema (accessible to any logged-in user)
- `/agenda/import` — XML agenda import
- `/staff/dashboard` — staff panel (role-gated)
- `/internal/config` — localhost-only config (reachable via XXE)

---

## HTML Comment Intelligence

From deploy-log comments embedded in every page:

```html
<!-- TODO: migrate session cookie to httponly=true — ticket #4201 — sophie -->
<!-- TODO: replace stdlib xml.etree with defusedxml — emilie reported issue -->
<!-- NOTE: /internal/config restricted to localhost only -->
<!-- v1.3.7 — sophie  — "fixed session handling" (she didn't, wil checked) -->
<!-- v1.3.4 — wil     — "disabled defusedxml temporarily" (that was 6 months ago) -->
<!-- v1.3.3 — wil     — "quick fix on file upload" (removed the whitelist entirely) -->
<!-- v1.2.0 — sophie  — initial platform. jwt secret was "42". wil changed it to "42network" -->
```

---

## Response Header Intelligence

Present on every response (including 403/405 errors):

x-powered-by: Python/3.11 FastAPI/0.104
server: uvicorn/0.24.0 Linux
x-pocketbase: http://localhost:8090
x-42-internal: campus=paris


On `/backup` (403 Forbidden):

x-backup-agent: rsync/3.2.7
x-backup-schedule: daily@03:00
x-backup-dest: localhost:/opt/pocketbase/pb_data
x-backup-exclude: data/private_notes.txt
x-last-backup: 2024-12-01T03:00:00Z


---

## Forum Intelligence

| Author | Post | Key Intel |
|--------|------|-----------|
| jdoe | "I'm not changing my password" | password "short and classic" — dictionary crackable |
| benjamin | PSA on forgot password | reset token = MD5(email), visible in URL |
| dorian | Darkly hints thread | XML import doesn't sanitize; check headers; creative filenames |
| wil | Base64 string | `dnFsaWRhd3lCbhZIbGp2dQ==` decodes to `vqlidawyBnHljvu` |
| thanos | NetPractice hint | 127.x.x.x loopback useful for internal tools |

---

## Console Easter Eggs (window.darkly42)

Shipped in production at `/static/js/console_eggs.js`:

| Command | Output |
|---------|--------|
| `darkly42.help()` | Lists all available commands |
| `darkly42.whoami()` | Prints raw session JWT from `document.cookie` |
| `darkly42.version()` | `{app: "1.3.7", pb: "0.22.4", note: "this object should not exist in production"}` |
| `darkly42.hint()` | Random hints pointing at vulnerabilities |
| `window.jwtDecode(token)` | Decodes JWT header + payload |

Boot log (printed to console on every page load):

[BOOT] PocketBase backend connected (127.0.0.1:8090)
[INFO] Campus: Wilcity | env: production | build 1.3.7
[WARN] 7 tickets assigned to @wil are past due (oldest: 2042-03).


---

## Discovered Credentials

| Account | Email | Password | Role | Source |
|---------|-------|----------|------|--------|
| jdoe | jdoe@student.42.tech | amine (original) → hacked123 | student | Forum self-admission + password reset |
| benjamin | benjamin@student.42.tech | hacked123 | student → god | Password reset |
| sophie | sophie@42.tech | akaddour | god | Prior intel |
| PocketBase admin | admin@42network.local | Darkly42Admin! | admin | XXE → /internal/config |

### Leaked Secrets
- **JWT Secret:** `42network`
- **Database path:** `/opt/pocketbase/pb_data`
- **Internal config endpoint:** `/internal/config` (localhost only, bypassed via XXE)

---

## Known User IDs

| User | ID |
|------|----|
| jdoe | z4p1cnx47mfy50f |
| sophie | enplwhu8jfo56oi |
| wil | k1asdfeditojrb4 |

---

## Session Cookie Analysis

set-cookie: session=<JWT>; Path=/; SameSite=lax


Missing:
- ❌ `HttpOnly` — JavaScript can read it via `document.cookie`
- ❌ `Secure` — sent over plain HTTP
- ⚠️ `SameSite=Lax` only — client-side only, not server-enforced

JWT payload structure:
```json
{"sub": "<user_id>", "login": "<username>", "role": "<role>", "exp": <timestamp>}
```
Signed with HS256 using secret `42network`.

---

## Initial Vulnerability Map (pre-exploitation)

| # | Vulnerability | Endpoint | Discovery Source |
|---|--------------|----------|-----------------|
| 1 | IDOR | `/profile/{id}` | Manual testing |
| 2 | Insecure Password Reset | `/reset-password` | Forum (benjamin) |
| 3 | Unrestricted File Upload | `/profile/me/settings` | HTML comment (v1.3.3) |
| 4 | Mass Assignment | `PATCH /api/profile` | HTML comment + /api/docs-internal |
| 5 | XXE/SSRF | `/agenda/import` | HTML comment (defusedxml disabled) |
| 6 | PocketBase Admin | `http://localhost:8090/_/` | XXE credential leak |
| 7 | Unprotected API | `/api/grades` | robots.txt |
| 8 | Stored XSS | `/forum` | HTML comment (emilie/wil exchange) |
| 9 | Path Traversal | `/projects/download` | Header + forum hint |
| 10 | CSRF | `/profile/me/settings` | Manual testing |
| + | Non-HttpOnly Cookie | session cookie | HTML comment (#4201) |
| + | Reflected XSS | `/newsletter` | HTML comment |
| + | Broken Access Control | `/api/docs-internal` | HTML comment |
| + | Header Info Disclosure | all responses | Manual observation |
| + | Debug Object in Production | `window.darkly42` | JS source |
| + | Open Redirect | `/redirect?next=` | HTML source |
| + | No Rate Limiting | `/login` | Manual testing |
| + | Missing Security Headers | all responses | Manual observation |
| + | JWT Forgery | session cookie | Derived from XXE leak |
| + | Base64 Obscurity | forum post | Forum (wil) |

