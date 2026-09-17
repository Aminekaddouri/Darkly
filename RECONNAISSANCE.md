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
