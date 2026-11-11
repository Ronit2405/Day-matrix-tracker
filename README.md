# Day Matrix Tracker

Apne din ko 30-minute slots mein plan karo, har slot ko **Urgent** (Y/N) aur **Important** (Y/N) mark karo, aur Eisenhower matrix se dekho ki aapka time kahan ja raha hai. Din ko Happy / Not happy mark karo — kuch dinon ka data hone par app aapka **best mix** nikaal ke dikhata hai.

**Live:** https://ronit2405.github.io/Day-matrix-tracker/

## Kya hai / kya nahi hai

- Ek hi file: `index.html` (HTML + CSS + chhota inline JS). Koi build step nahi.
- Koi login, signup, form, OTP, payment field — **nahi**.
- Koi external script, CDN, font, analytics, iframe, pixel — **kuch nahi**. Zero network calls.
- Data sirf aapke browser ke `localStorage` (key `dmt_v1`) mein. Clear karoge to chala jayega — **Help & data** tab se backup lo.
- Strict CSP: page external kuch load hi nahi kar sakta.

## Files

| File | Kaam |
|---|---|
| `index.html` | Poora app |
| `about.html` | About & privacy page |
| `sitemap.xml`, `robots.txt` | Search engines ke liye |
| `404.html` | Custom not-found page |
| `netlify.toml` | Alternate deploy (Netlify) ke liye config |
| `FIX-BROWSER-WARNING.md` | Chrome ke "Dangerous site" warning ka diagnosis + fix plan |

## Local par chalana

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Safety / privacy (verified 11 Nov 2026)

- `index.html`, `about.html`, `404.html` mein **koi external request nahi** — na script, na CDN, na font, na analytics, na iframe.
- **Koi `<form>` nahi**, koi password / email / OTP / payment / card / file input nahi.
- `eval` / `atob` / obfuscated JS: **nahi**. Redirect, download, geolocation: **nahi**.
- Strict CSP (`default-src 'none'`) — page bahar se kuch load ya POST hi nahi kar sakta.
- Data sirf `localStorage` (key `dmt_v1`). Restore/load par data validate + sanitize hota hai.

## Warning ke baare mein

Agar Chrome "Dangerous site / phishing" dikhata hai: maine check kiya — Google Safe Browsing endpoint `ronit2405.github.io` ke liye **code `3`** ("some pages on this site are unsafe") return karta hai, iss path ke liye bhi wahi. Yani verdict **host-level** hai, is app ka nahi, aur **sirf Google hi review ke baad hata sakta hai**. Poora verified diagnosis, evidence table, ready-to-send report text aur step-by-step plan: [`FIX-BROWSER-WARNING.md`](FIX-BROWSER-WARNING.md).

**Alag dhyan dene wali baat:** is waqt `https://ronit2405.github.io/Day-matrix-tracker/` khud "There isn't a GitHub Pages site here" de raha hai (build green hone ke bawajood). Live URL pehle theek karo, phir review request bhejo — detail `FIX-BROWSER-WARNING.md` section 6 mein.
