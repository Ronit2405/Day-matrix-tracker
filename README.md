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

## Note

Agar Chrome is site par "Dangerous site / phishing" warning dikhata hai, to wajah is repo ka code nahi hai — Google Safe Browsing ka `ronit2405.github.io` host-level verdict hai, jo GitHub Pages ke shared host ki reputation se aata hai. Poora diagnosis aur fix steps: [`FIX-BROWSER-WARNING.md`](FIX-BROWSER-WARNING.md).
