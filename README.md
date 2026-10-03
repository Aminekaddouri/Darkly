# Darkly v2 — Web Security Audit

**42 School — Holy Graph Curriculum**

A comprehensive security audit of Darkly v2, a deliberately vulnerable
web application built on FastAPI + PocketBase.

## Quick Start

Read [walkthrough.md](walkthrough.md) for the complete audit narrative,
exploitation chain, and all findings.

## Structure

exploits/

├── 01_idor/ # FLAG{1d0r_ur_pr0f1l3_1s_m1n3}

├── 02_insecure_password_reset/ # FLAG{r3s3t_t0k3n_w4s_just_md5_lol}

├── 03_unrestricted_upload/ # FLAG{unr3str1ct3d_upl0ad_g0_brrr}

├── 04_mass_assignment/ # FLAG{just_p4tch_y0ur_0wn_r0l3_lol}

├── 05_xxe_ssrf/ # FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3}

├── 06_pocketbase_admin/ # FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y}

├── BONUS_01_unprotected_api/ # FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck}

├── BONUS_02_stored_xss/ # FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l}

├── BONUS_03_path_traversal/ # FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn}

├── BONUS_04_csrf/ # FLAG{csrf_4ny_0r1g1n_1s_w3lc0m3}

├── VULN_01_non_httponly_cookie/ # No flag

├── VULN_02_reflected_xss_newsletter/ # No flag

├── VULN_03_unauthenticated_docs_internal/ # No flag

├── VULN_04_header_info_disclosure/ # No flag

├── VULN_05_console_debug_disclosure/ # No flag

├── VULN_06_open_redirect/ # No flag

├── VULN_07_no_rate_limit_weak_password/ # No flag

├── VULN_08_missing_security_headers/ # No flag

├── VULN_09_jwt_forgery/ # No flag

└── VULN_10_base64_obscurity/ # No flag


Each folder contains:
- `explanation.md` — vulnerability description, impact, and remediation
- `exploit.sh` — automated reproduction script
- `flag` — the captured flag (flag-bearing breaches only)

## Stats

| | Count |
|---|---|
| Total flags found | 10/10 |
| Mandatory flags | 6/6 |
| Bonus flags | 4/4 |
| Total vulnerabilities documented | 20 |
| Flag-bearing breaches | 10 |
| No-flag vulnerabilities | 10 |

## Target

- **App:** http://localhost:4942
- **Backend:** http://localhost:8090 (PocketBase)
- **Stack:** FastAPI 0.104 / Python 3.11 / PocketBase 0.22.4

## Author

AmineKaddouri — 42 School (Holy Graph)  
September–October 2026
