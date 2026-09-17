# Darkly v2 - Reconnaissance Report

## 1. Target Information

| Property | Value |
|----------|-------|
| URL | http://localhost:4942 |
| Backend | PocketBase (http://localhost:8090/_/) |
| Version | 1.3.7-prod |
| Campus | wilcity |
| Stack | FastAPI 0.104 / Python 3.11 / PocketBase 0.22.4 / uvicorn 0.24.0 |
| JWT Secret | "42network" (changed from "42") |

---

## 2. Attack Surface

### Disallowed Paths (from robots.txt)
- `/admin` - Admin panel
- `/staff` - Staff area  
- `/internal` - Internal configuration
- `/backup` - Backup endpoint
- `/api/grades` - Grade API endpoint
- `/_/` - PocketBase admin
- `/static/uploads/` - File uploads directory

### Additional Accessible Pages
- `/projects` - Project listing with downloads
- `/forum` - Student forum with posts
- `/newsletter` - Email subscription
- `/login` - Authentication form
- `/reset-password` - Password reset flow
- `/api/docs-internal` - API schema (no auth)

---

## 3. Identified Vulnerabilities

### Vuln #1: Insecure Password Reset (CRITICAL)
**Endpoint:** `/reset-password`  
**Discovery:** Forum post by benjamin  
**Issue:** Reset token = MD5(email address) - trivially predictable  
**Exploitation:**
```bash
echo -n "jdoe@student.42.tech" | md5sum
# Result is the reset token!
```
**Impact:** Can reset any account, including admin accounts

---

### Vuln #2: Weak Password - Dictionary Crackable
**Target:** jdoe@student.42.tech  
**Discovery:** Forum post by jdoe  
**Issue:** jdoe admits password is "short, it's classic"  
**Exploitation:** Brute force with common password lists  
**Impact:** Account compromise via credential guessing

---

### Vuln #3: Mass Assignment - Privilege Escalation
**Endpoint:** `PATCH /api/profile`  
**Discovery:** HTML comment in /staff page  
**Issue:** Accepts "role" parameter with values: visitor, student, cadet, staff, god  
**Exploitation:** Change role from student → cadet/staff/god  
**Impact:** Full privilege escalation to admin

---

### Vuln #4: Path Traversal - File Download
**Endpoint:** `/projects/download?file=`  
**Discovery:** HTML comment + forum hints  
**Issue:** No path validation - "serves any file you ask for"  
**Exploitation:** `../../../etc/passwd` or `../../../opt/pocketbase/pb_data/`  
**Impact:** Arbitrary file access including sensitive configs

---

### Vuln #5: XXE/SSRF - Unsafe XML Parser
**Location:** XML upload/import functionality  
**Discovery:** HTML comments mention defusedxml disabled  
**Issue:** Using stdlib ET instead of defusedxml (6+ months without fix)  
**Exploitation:** XXE payload in XML uploads  
**Impact:** SSRF, file disclosure, potential RCE

---

### Vuln #6: Non-HttpOnly Cookies
**Cookie:** session  
**Discovery:** HTML comment about ticket #4201  
**Issue:** httponly=false - accessible via JavaScript  
**Exploitation:** XSS payload to steal session cookie  
**Impact:** Session hijacking

---

### Vuln #7: Reflected XSS - Newsletter Email
**Endpoint:** `/newsletter` subscription form  
**Discovery:** HTML comments + working PoC  
**Issue:** Email field echoed unescaped in banner  
**Exploitation:** `<script>alert(1)</script>` in email field  
**Impact:** Reflected XSS in banner

---

### Vuln #8: Stored XSS - Newsletter Database
**Location:** Newsletter subscriptions  
**Discovery:** Forum post - "someone subscribed with a <script> tag last week. wil: bold of them. it worked."  
**Issue:** Scripts stored in database and executed  
**Impact:** Persistent XSS affecting all viewers

---

### Vuln #9: Unrestricted File Upload
**Location:** File upload functionality  
**Discovery:** HTML comment - "removed the whitelist entirely"  
**Issue:** No file type validation or restrictions  
**Exploitation:** Upload .php/.sh/.exe → execute code  
**Impact:** Remote Code Execution (RCE)

---

### Vuln #10: Information Disclosure via API Schema
**Endpoint:** `/api/docs-internal`  
**Discovery:** HTML comment - "still accessible without staff check"  
**Issue:** No authentication required on schema endpoint  
**Leaks:** API structure, role values, allowed fields  
**Impact:** Reveals exploitation vectors

---

## 4. Additional Intelligence

### From Forum Posts
- **Base64 Credential:** dnFsaWRhd3lCbhZIbGp2dQ== decodes to: vqlidawyBnHljvu (purpose unknown)
- **Response Headers:** Contain leaked information beyond intended exposure
- **Creative Filenames:** Path traversal works with specific filename patterns
- **Session Management:** Still has bugs despite claimed "fixes"

### From HTML Comments
- **Deploy History:** Shows 6+ versions with accumulated vulnerabilities
- **Hardcoded Values:** Students online counter hardcoded (info disclosure)
- **Past Vulnerabilities:** SQL injection in v1 (claims to be fixed)
- **Dev Notes:** wil takes 6+ months to fix known issues

### From Console Easter Eggs
- Sophie is "Level 42" - the final boss
- Hints point to: robots.txt, base64, HTML comments, forum, maintenance pages
- JWT decoder available: `window.jwtDecode(token)`
- Telemetry sends user-agent to `/api/telemetry/heartbeat`

---

## 5. Exploitation Chain

### Initial Access Path
1. Calculate MD5 of target email: `md5("jdoe@student.42.tech")`
2. Use MD5 hash as reset token at `/reset-password`
3. Gain access to jdoe account
4. Retrieve account recovery code from profile

### Privilege Escalation
1. Send `PATCH /api/profile` with `role=cadet` or `role=god`
2. Escalate to staff/admin privileges
3. Access `/staff`, `/admin`, `/internal/config` endpoints
4. Access `/api/grades` for sensitive data

### Full Compromise
1. XXE injection via XML upload (if file upload available)
2. Stored XSS via forum/newsletter for bot session hijacking
3. Path traversal to access PocketBase data files
4. RCE via file upload

---

## 6. Summary

**Total Vulnerabilities:** 10 critical security flaws  
**Easiest Entry Point:** Insecure password reset (MD5 token)  
**Fastest Escalation:** Mass assignment via PATCH /api/profile  
**Most Severe:** Unrestricted file upload (RCE potential)  

**Reconnaissance Status:** COMPLETE ✅  
**Ready for Exploitation:** YES ✅

