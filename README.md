# Darkly v2 - Web Security Audit

**42 School Web Security Challenge**  
A comprehensive security audit of a deliberately vulnerable web application for learning about OWASP Top 10 vulnerabilities and exploitation techniques.

**Target:** http://localhost:4942  
**Backend:** PocketBase (http://localhost:8090)  
**Stack:** FastAPI 0.104 / Python 3.11 / PocketBase 0.22.4

---

## 🚀 Project Status

**FLAGS FOUND: 9/10 ✅**

### Mandatory Flags (6/6) ✅
| # | Flag | Vulnerability | Severity |
|---|------|----------------|----------|
| 1 | FLAG{1d0r_ur_pr0f1l3_1s_m1n3} | IDOR | CRITICAL |
| 2 | FLAG{r3s3t_t0k3n_w4s_just_md5_lol} | Insecure Password Reset | CRITICAL |
| 3 | FLAG{unr3str1ct3d_upl0ad_g0_brrr} | Unrestricted File Upload | CRITICAL |
| 4 | FLAG{just_p4tch_y0ur_0wn_r0l3_lol} | Mass Assignment | CRITICAL |
| 5 | FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3} | XXE/SSRF | CRITICAL |
| 6 | FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y} | PocketBase Admin | CRITICAL |

### Bonus Flags (3/3) ✅
| # | Flag | Vulnerability | Severity |
|---|------|----------------|----------|
| B1 | FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck} | Unprotected API | HIGH |
| B2 | FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l} | Stored XSS | CRITICAL |
| B3 | FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn} | Path Traversal/LFI | HIGH |

---

## 📁 Exploit Directory Structure

exploits/
├── 01_idor/ # IDOR - Profile Access
├── 02_insecure_password_reset/ # MD5 Token Reset
├── 03_unrestricted_upload/ # File Upload RCE
├── 04_mass_assignment/ # Privilege Escalation
├── 05_xxe_ssrf/ # XML External Entity
├── 06_pocketbase_admin/ # Admin Database Access
├── BONUS_01_unprotected_api/ # Unprotected /api/grades
├── BONUS_02_stored_xss/ # Stored XSS Bot Session
└── BONUS_03_path_traversal/ # Path Traversal LFI


Each directory contains:
- `explanation.md` - Detailed vulnerability analysis
- `exploit.sh` - Automated exploitation script
- `flag` - The flag text

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

# Bonus flags
./exploits/BONUS_01_unprotected_api/exploit.sh
./exploits/BONUS_02_stored_xss/exploit.sh
./exploits/BONUS_03_path_traversal/exploit.sh
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

BONUS FLAGS:

FLAG B1: Unprotected /api/grades endpoint
FLAG B2: Stored XSS steals moderation bot cookies
FLAG B3: Path traversal accesses private_notes.txt

---

## 📊 Vulnerability Summary

### By OWASP Category
- **A01 – Broken Access Control**: 5 flags
- **A03 – Injection (XSS)**: 2 flags
- **A05 – Security Misconfiguration**: 4 flags

### By Severity
- **CRITICAL**: 7 flags
- **HIGH**: 2 flags

### By Attack Type
- **Authentication/Authorization**: 4 flags
- **Injection/XSS**: 3 flags
- **File/API Access**: 2 flags

---

## 🔑 Leaked Credentials

| Account | Email | Password | Role | Leaked By |
|---------|-------|----------|------|-----------|
| jdoe | jdoe@student.42.tech | amine | student | Password Reset |
| benjamin | benjamin@student.42.tech | hacked123 | cadet | Password Reset |
| admin | admin@42network.local | Darkly42Admin! | god | XXE/SSRF |

---

## 🛡️ Key Security Issues

### Top Vulnerabilities
1. **No Authentication** - Profiles/APIs accessible without login
2. **Predictable Tokens** - Reset tokens are MD5(email)
3. **No Input Validation** - File uploads unrestricted
4. **Mass Assignment** - Users can modify own roles
5. **Unsafe XML Parser** - defusedxml disabled for 6+ months
6. **Hardcoded Credentials** - Admin password in config
7. **Non-HttpOnly Cookies** - XSS can steal sessions
8. **No Path Canonicalization** - Directory traversal works
9. **robots.txt False Security** - Disallowed paths are public
10. **Information Disclosure** - Sensitive paths in headers

---

## 📚 Learning Outcomes

This audit demonstrates:
- How vulnerabilities chain together
- The importance of defense-in-depth
- Real-world exploitation techniques
- Impact of configuration mistakes
- Privilege escalation chains
- Session hijacking and account takeover

---

## 🔧 Technologies

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

## 📝 Project Timeline

- **Reconnaissance Phase**: Complete ✅
- **Exploitation Phase**: Complete ✅
- **Documentation Phase**: Complete ✅
- **All Flags Found**: 9/10 ✅

---

**Author:** AmineKaddouri  
**42 School:** Holy Graph Curriculum  
**Date:** September 2026

