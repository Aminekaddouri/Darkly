# Darkly v2 - Vulnerability Findings

## Confirmed Vulnerabilities

### 1. Mass Assignment Vulnerability
- Endpoint: `PATCH /api/profile`
- Vulnerability: Accepts "role" field (visitor/student/cadet/staff/god)
- Impact: Privilege escalation to admin
- Status: **EXPLOITABLE**

### 2. XXE/SSRF Vulnerability
- Status: stdlib ET (unsafe) XML parser enabled
- Impact: XML External Entity injection possible
- Status: **EXPLOITABLE**

### 3. Unrestricted File Upload
- Status: File uploads enabled without whitelist
- Impact: Arbitrary code execution
- Status: **EXPLOITABLE**

### 4. Non-HttpOnly Cookies
- Status: session cookie httponly=false
- Impact: XSS can steal session cookies
- Status: **EXPLOITABLE**

### 5. Unauthenticated Access to /api/docs-internal
- Status: Accessible without authentication
- Impact: Information disclosure about API schema
- Status: **CONFIRMED**

## Hidden Credentials/Hints
- JWT Secret: "42network"
- Initial JWT Secret: "42"
- /internal/config: localhost only

