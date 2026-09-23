# Darkly v2 - Web Security Audit

**42 School Web Security Challenge**  
A comprehensive security audit of a deliberately vulnerable web application demonstrating OWASP Top 10 vulnerabilities and exploitation techniques.

**Target:** http://localhost:4942  
**Backend:** PocketBase (http://localhost:8090)  
**Stack:** FastAPI 0.104 / Python 3.11 / PocketBase 0.22.4

---

## 🎉 Project Status

**ALL FLAGS FOUND: 10/10 ✅**

### Mandatory Flags (6/6) ✅
| # | Flag | Vulnerability | Severity |
|---|------|----------------|----------|
| 1 | FLAG{1d0r_ur_pr0f1l3_1s_m1n3} | IDOR | CRITICAL |
| 2 | FLAG{r3s3t_t0k3n_w4s_just_md5_lol} | Insecure Password Reset | CRITICAL |
| 3 | FLAG{unr3str1ct3d_upl0ad_g0_brrr} | Unrestricted File Upload | CRITICAL |
| 4 | FLAG{just_p4tch_y0ur_0wn_r0l3_lol} | Mass Assignment | CRITICAL |
| 5 | FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3} | XXE/SSRF | CRITICAL |
| 6 | FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y} | PocketBase Admin | CRITICAL |

### Bonus Flags (4/4) ✅
| # | Flag | Vulnerability | Severity |
|---|------|----------------|----------|
| B1 | FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck} | Unprotected API | HIGH |
| B2 | FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l} | Stored XSS | CRITICAL |
| B3 | FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn} | Path Traversal/LFI | HIGH |
| 10 | FLAG{csrf_4ny_0r1g1n_1s_w3lc0m3} | CSRF | MEDIUM |

---

## 📁 Exploit Directory

exploits/
├── 01_idor/ # IDOR - Profile Access
├── 02_insecure_password_reset/ # MD5 Token Reset
├── 03_unrestricted_upload/ # File Upload RCE
├── 04_mass_assignment/ # Privilege Escalation
├── 05_xxe_ssrf/ # XML External Entity
├── 06_pocketbase_admin/ # Admin Database Access
├── BONUS_01_unprotected_api/ # Unprotected /api/grades
├── BONUS_02_stored_xss/ # Stored XSS Bot Session
├── BONUS_03_path_traversal/ # Path Traversal LFI
└── 10_csrf/ # CSRF Origin Bypass


---

## 🎯 Quick Start

### Run All Exploits

```bash
# Mandatory flags
./exploits/01_idor/exploit.sh
./exploits/02_insecure_password_reset/exploit.sh
./exploits/03_unrestricted_upload/exploit.sh
./exploits/04_mass_assignment/exploit.sh
./exploits/05_xxe_ssrf/exploit.sh
./exploits/06_pocketbase_admin/exploit.sh

# Bonus + Flag #10
./exploits/BONUS_01_unprotected_api/exploit.sh
./exploits/BONUS_02_stored_xss/exploit.sh
./exploits/BONUS_03_path_traversal/exploit.sh
./exploits/10_csrf/exploit.sh
```

---

## 🔗 Exploitation Chain

┌─────────────────────────────────────────────────────┐
│ FLAG #1: IDOR │
│ Access any user's profile without authentication │
└────────────────────┬────────────────────────────────┘
│
┌────────────────────▼────────────────────────────────┐
│ FLAG #2: Insecure Password Reset (MD5 Token) │
│ Reset any account using MD5(email) as token │
└────────────────────┬────────────────────────────────┘
│
┌────────────────────▼────────────────────────────────┐
│ FLAG #3: Unrestricted File Upload │
│ Upload any file type without validation │
└────────────────────┬────────────────────────────────┘
│
┌────────────────────▼────────────────────────────────┐
│ FLAG #4: Mass Assignment - Privilege Escalation │
│ PATCH /api/profile to change role to god/admin │
└────────────────────┬────────────────────────────────┘
│
┌────────────────────▼────────────────────────────────┐
│ FLAG #5: XXE/SSRF - Internal Config Access │
│ XML parser leaks JWT secret & PocketBase credentials│
└────────────────────┬────────────────────────────────┘
│
┌────────────────────▼────────────────────────────────┐
│ FLAG #6: PocketBase Admin Access │
│ Complete database compromise via admin console │
└─────────────────────────────────────────────────────┘

BONUS FLAGS & FLAG #10:

FLAG B1: Unprotected /api/grades endpoint
FLAG B2: Stored XSS steals moderation bot cookies
FLAG B3: Path traversal accesses private_notes.txt
FLAG 10: CSRF modifies profile without validation

---

## 📊 Vulnerability Summary

### By OWASP Category
- **A01 – Broken Access Control**: 6 flags
- **A03 – Injection (XSS)**: 2 flags
- **A05 – Security Misconfiguration**: 5 flags

### By Severity
- **CRITICAL**: 7 flags
- **HIGH**: 2 flags
- **MEDIUM**: 1 flag

### By Attack Type
- **Authentication/Authorization**: 4 flags
- **Injection/XSS**: 2 flags
- **File/API Access**: 2 flags
- **State Manipulation**: 2 flags

---

## 🔑 Key Findings

### Critical Vulnerabilities
1. **No Authentication** - Profiles/APIs accessible without login
2. **Predictable Tokens** - Reset tokens are MD5(email)
3. **No Input Validation** - File uploads completely unrestricted
4. **Mass Assignment** - Users can modify their own roles
5. **Unsafe XML Parser** - defusedxml disabled, XXE possible
6. **Hardcoded Admin Credentials** - Password in config file
7. **Non-HttpOnly Cookies** - XSS can steal sessions
8. **No Path Canonicalization** - Directory traversal works
9. **robots.txt False Security** - Disallowed paths are public
10. **No CSRF Protection** - State changes accept any origin

### Leaked Credentials
| Account | Email | Password | Role | Leaked By |
|---------|-------|----------|------|-----------|
| jdoe | jdoe@student.42.tech | hacked123 | student | Password Reset |
| benjamin | benjamin@student.42.tech | hacked123 | cadet/staff/god | Password Reset |
| admin | admin@42network.local | Darkly42Admin! | god | XXE/SSRF |

### Leaked Secrets
- JWT Secret: `42network`
- Database location: `/opt/pocketbase/pb_data`
- Internal endpoints: `/internal/config`, `/api/grades`

---

## 🛡️ Defense Layers Missing

- ❌ No input validation
- ❌ No output encoding
- ❌ No authentication checks
- ❌ No authorization enforcement
- ❌ No CSRF tokens
- ❌ No Content Security Policy
- ❌ No HttpOnly/Secure cookies
- ❌ No rate limiting
- ❌ No audit logging
- ❌ No secret management

---

## 📚 Learning Outcomes

This audit demonstrates:
- How vulnerabilities chain together for maximum impact
- The importance of defense-in-depth
- Real-world exploitation techniques and tools
- Impact of configuration mistakes and shortcuts
- Complete privilege escalation and account takeover
- Session hijacking and credential theft
- State manipulation and CSRF

---

## 🔧 Technologies & Versions

- **Language:** Python 3.11
- **Framework:** FastAPI 0.104
- **Database:** PocketBase 0.22.4
- **Server:** uvicorn 0.24.0
- **Frontend:** HTML/CSS/JavaScript

---

## 📖 References

- OWASP Top 10 2021
- OWASP Testing Guide
- CWE/SANS Top 25
- Security Testing Best Practices

---

## 📝 Project Completion

- ✅ **Reconnaissance Phase**: Complete
- ✅ **Exploitation Phase**: Complete
- ✅ **All 10 Flags Found**: Complete
- ✅ **Documentation**: Complete
- ✅ **Exploit Scripts**: Complete

---

## 🎯 Summary

**Total Vulnerabilities Found:** 10  
**Critical Issues:** 7  
**High Issues:** 2  
**Medium Issues:** 1  

**Impact:** Complete Application Compromise
- Full database access
- All user accounts compromised
- Complete state manipulation
- Privilege escalation to administrator
- Information disclosure of all secrets

---

**Author:** AmineKaddouri  
**42 School:** Holy Graph Curriculum  
**Date:** September 2026  
**Status:** COMPLETE ✅

