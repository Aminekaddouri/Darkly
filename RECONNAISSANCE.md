# Darkly v2 - Reconnaissance Report

## Target Information
- **URL:** http://localhost:4942
- **Backend:** PocketBase (http://localhost:8090/_/)
- **Version:** 1.3.7-prod
- **Campus:** wilcity

## Attack Surface - Disallowed Paths (from robots.txt)
1. `/admin` - Admin panel
2. `/staff` - Staff area
3. `/internal` - Internal configuration
4. `/backup` - Backup endpoint
5. `/api/grades` - Grade API endpoint
6. `/_/` - PocketBase admin
7. `/static/uploads/` - File uploads directory

## Vulnerabilities Hints from HTML Comments
- **SQL Injection:** "v1 had SQL in login form"
- **XXE/SSRF:** "defusedxml disabled temporarily"
- **File Upload:** "removed the whitelist entirely"
- **XSS:** "session cookie httponly=false"
- **Information Disclosure:** "students online counter hardcoded"

## Technology Stack
- FastAPI 0.104 (Python 3.11)
- PocketBase 0.22.4
- uvicorn 0.24.0

## Key Credentials/Hints
- JWT Secret: "42network"
- Internal config at /internal/config (localhost only)


## Detailed Vulnerability Analysis (from HTML Comments)

### 1. Reflected XSS - Newsletter Email
**Source Comment:**

- **Location:** `/newsletter` subscription form
- **Parameter:** email field (echoed unescaped)
- **Attack:** `<script>alert(1)</script>` in email
- **Impact:** Reflected XSS on banner

### 2. Stored XSS - Newsletter Database
- **Location:** Newsletter subscriptions stored in database
- **Trigger:** Viewing subscriber list or history
- **Proof:** Confirmed working - "someone subscribed with a <script> tag"
- **Impact:** Persistent XSS affecting all viewers

### 3. Path Traversal - File Download
**Source Comment:**

- **Endpoint:** `/projects/download?file=`
- **Attack Vector:** `../../../etc/passwd` or `../../../opt/pocketbase/pb_data/`
- **Status:** No validation - "serves any file you ask for"

### 4. Mass Assignment - Privilege Escalation
**Source Comment:**

- **Endpoint:** `PATCH /api/profile`
- **Parameter:** `role`
- **Valid Values:** visitor, student, cadet, staff, god
- **Attack:** Change role to escalate privileges
- **Impact:** Full admin access

### 5. XXE/SSRF - Unsafe XML Parser
**Source Comment:**

- **Parser:** stdlib ET (vulnerable to XXE)
- **Status:** defusedxml disabled "temporarily" 6 months ago
- **Attack Vector:** XXE payload in XML uploads
- **Impact:** SSRF, file disclosure, RCE potential

### 6. Non-HttpOnly Cookies - XSS Risk
**Source Comment:**

- **Cookie:** session (httponly=false)
- **Risk:** Accessible via JavaScript/XSS
- **Attack:** XSS payload steals session cookie
- **Impact:** Session hijacking

### 7. Unrestricted File Upload
**Source Comment:**

- **Status:** File whitelist completely removed
- **Attack:** Upload .php/.sh/.exe → execute
- **Impact:** Remote Code Execution (RCE)
- **Endpoint:** `/projects/download` (used for upload as well)

### 8. SQL Injection - Login Form (Status Unclear)
**Source Comment:**

- **Location:** `/login` form
- **Status:** Claims fixed but "probably" suggests uncertainty
- **Test:** Standard SQLi payloads in username/password

### 9. Insecure Password Reset
- **Endpoint:** `/reset-password`
- **Hint:** Token generation likely MD5-based (weak)
- **Attack:** Predict or brute-force reset tokens

### 10. Information Disclosure via API Schema
**Source Comment:**

- **Endpoint:** `/api/docs-internal`
- **Access:** No authentication required (bug)
- **Leaks:** API schema, role values, field names
- **Impact:** Reveals exploitation vectors

## Additional Hints
- **FAQ Hidden Note:** "faq_darkly.pdf has a note at the bottom" (file download test)
- **Deploy History:** Shows progression of vulnerabilities introduced/removed
- **Hardcoded Values:** "students online counter has been hardcoded since launch" (info disclosure)

## Summary
- **Total Vulnerabilities Found:** 10 major vectors
- **Exploitation Chain:** Mass Assignment → Privilege Escalation → Admin Access
- **Critical Flaws:** XXE (defusedxml), File Upload (no whitelist), XSS (unescaped), Cookies (not httponly)
- **Information Leakage:** Multiple endpoints expose sensitive data

## Forum Intelligence - Critical Exploitation Hints

### Forum Disclosure 1: jdoe Password Weakness
**Post:** "jdoe here 🔐 and no, I'm not changing my password"
- jdoe admits password is "short, it's classic, it rocks"
- Attack Vector: Dictionary/brute force attack
- Password likely in common wordlists

### Forum Disclosure 2: INSECURE PASSWORD RESET (CRITICAL!)
**Post:** "PSA: the 'forgot password' flow is... not great?"
**By:** benjamin

**Exploitation:**
```bash
# For jdoe@student.42.tech:
echo -n "jdoe@student.42.tech" | md5sum
# This is the reset token!
```

### Forum Disclosure 3: XXE Vulnerability Confirmed
**Post:** "Darkly project — hints thread"
**Hints:**
- XML import doesn't sanitize anything
- Check response headers for leaked info
- Resources section vulnerable to path traversal with creative filenames
- Network team post has encoded string (base64?)

### Forum Disclosure 4: Base64 Encoded Credential
**Post:** "Scheduled maintenance — this Saturday 2AM CET"
**By:** wil (Network Ops)


### Forum Disclosure 5: Loopback Access Hint
**Post:** "NetPractice level 9 — gateway trick"
- 127.x.x.x loopback space "interesting if you're ever on internal tools"
- Suggests localhost-only endpoints might be exploitable

### Forum Disclosure 6: Session Handling Issues
**Post:** "Platform update — v1.3.7 deployed"
- Session handling fixes listed as "ongoing"
- Implies session management still buggy

## Exploitation Chain (From Forum Intel)

1. **Option A - Weak Password Brute Force**
   - Target: jdoe@student.42.tech
   - Method: Dictionary attack (password is "short, classic")
   - Tool: hydra, hashcat, or manual curl

2. **Option B - Insecure Password Reset (FASTER)**
   - Calculate: MD5("jdoe@student.42.tech") = reset_token
   - Use reset token to gain access
   - Retrieve account recovery code from profile
   - Escalate privileges

3. **Privilege Escalation Chain**
   - Once logged in as jdoe
   - PATCH /api/profile with role=cadet/staff/god
   - Access /internal/config (localhost requirement)
   - Access /api/grades, /admin, /staff endpoints
   - Full platform compromise

## Unknowns to Investigate
- Purpose of base64 string (vqlidawyBnHljvu)
- Exact jdoe password (for brute force comparison)
- Which "creative filenames" work for path traversal
- Exact XXE payload structure needed

