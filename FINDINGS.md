# Darkly v2 - Confirmed Vulnerabilities

**See RECONNAISSANCE.md for detailed discovery notes, exploitation vectors, and intelligence sources.**

---

## Vulnerability Summary

### 1. Insecure Password Reset (CRITICAL)
- **Endpoint:** `/reset-password`
- **Issue:** Reset token = MD5(email address)
- **Impact:** Can reset ANY account
- **Status:** ✅ EXPLOITABLE
- **Severity:** CRITICAL

### 2. Mass Assignment - Privilege Escalation
- **Endpoint:** `PATCH /api/profile`
- **Parameter:** role (values: visitor/student/cadet/staff/god)
- **Impact:** Escalate to admin
- **Status:** ✅ EXPLOITABLE
- **Severity:** CRITICAL

### 3. Path Traversal - File Download
- **Endpoint:** `/projects/download?file=`
- **Issue:** No path validation
- **Impact:** Access arbitrary files including /opt/pocketbase/pb_data/
- **Status:** ✅ EXPLOITABLE
- **Severity:** CRITICAL

### 4. XXE/SSRF - Unsafe XML Parser
- **Location:** XML upload/import
- **Issue:** Using stdlib ET (defusedxml disabled)
- **Impact:** SSRF, file disclosure, RCE potential
- **Status:** ✅ EXPLOITABLE
- **Severity:** HIGH

### 5. Unrestricted File Upload
- **Location:** File upload endpoint
- **Issue:** No whitelist, no type validation
- **Impact:** Upload .php/.sh → RCE
- **Status:** ✅ EXPLOITABLE
- **Severity:** CRITICAL

### 6. Non-HttpOnly Cookies
- **Cookie:** session
- **Issue:** httponly=false
- **Impact:** XSS can steal session token
- **Status:** ✅ EXPLOITABLE
- **Severity:** HIGH

### 7. Reflected XSS - Newsletter
- **Endpoint:** `/newsletter` subscription
- **Parameter:** email field (unescaped)
- **Impact:** Reflected XSS in banner
- **Status:** ✅ EXPLOITABLE
- **Severity:** MEDIUM

### 8. Stored XSS - Newsletter Database
- **Location:** Newsletter subscriptions
- **Issue:** Scripts stored and executed
- **Impact:** Persistent XSS for all viewers
- **Status:** ✅ EXPLOITABLE
- **Severity:** HIGH

### 9. Information Disclosure - API Schema
- **Endpoint:** `/api/docs-internal`
- **Issue:** No authentication required
- **Impact:** Leaks API structure and role values
- **Status:** ✅ CONFIRMED
- **Severity:** MEDIUM

### 10. Weak Passwords - Dictionary Attack
- **Target:** jdoe@student.42.tech
- **Issue:** Password is "short, it's classic"
- **Impact:** Account compromise via brute force
- **Status:** ✅ EXPLOITABLE
- **Severity:** HIGH

---

## Key Credentials

- **JWT Secret:** "42network"
- **Initial Secret:** "42"
- **PocketBase Admin URL:** http://localhost:8090/_/
- **Internal Config:** /internal/config (localhost only)

---

## Exploitation Priority

1. **Fastest:** Insecure Password Reset (MD5 token)
2. **Easiest:** Mass Assignment (PATCH /api/profile)
3. **Most Severe:** File Upload (RCE)
4. **Alternative:** Weak Password Brute Force

---

## Total Status

- **Total Vulnerabilities:** 10
- **Critical:** 4
- **High:** 4
- **Medium:** 2
- **All Exploitable:** ✅ YES

