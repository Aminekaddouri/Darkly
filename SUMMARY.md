# Darkly v2 - Quick Summary

## All 10 Flags
FLAG{1d0r_ur_pr0f1l3_1s_m1n3} - IDOR
FLAG{r3s3t_t0k3n_w4s_just_md5_lol} - Insecure Password Reset
FLAG{unr3str1ct3d_upl0ad_g0_brrr} - Unrestricted File Upload
FLAG{just_p4tch_y0ur_0wn_r0l3_lol} - Mass Assignment
FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3} - XXE/SSRF
FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y} - PocketBase Admin
FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck} - Unprotected API
FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l} - Stored XSS
FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn} - Path Traversal
FLAG{csrf_4ny_0r1g1n_1s_w3lc0m3} - CSRF

## Quick Run

```bash
cd ~/Documents/1337/Darkly-V2
for dir in exploits/*/; do
  echo "Running $dir..."
  ./$dir/exploit.sh
done
```

## Key Credentials

- **jdoe:** jdoe@student.42.tech / hacked123
- **benjamin:** benjamin@student.42.tech / hacked123
- **PocketBase Admin:** admin@42network.local / Darkly42Admin!

## Key Secrets

- **JWT Secret:** 42network
- **Database:** /opt/pocketbase/pb_data

