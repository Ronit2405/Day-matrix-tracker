# Chrome / Google Safe Browsing warning — verified diagnosis and action plan

**TL;DR (Hinglish):** Is repo ke code mein kuch galat nahi mila — maine poora audit kiya. Warning **host-level** hai: Google Safe Browsing ne `ronit2405.github.io` (poore shared host) par verdict `3` = *"some pages on this site are unsafe"* laga rakha hai. Is page ko badal ke warning nahi hata sakte — **Sirf Google hi hata sakta hai**, review request se. Neeche ready-to-send report text hai.

---

## 1. Verified verdict (checked 11 Nov 2026)

Google's public Safe Browsing status endpoint, raw responses:

| Query | Raw response | Code | Meaning |
|---|---|---|---|
| `example.com` | `[["sb.ssr",1,false,false,false,...]]` | `1` | No unsafe content (control) |
| `testsafebrowsing.appspot.com` (Google's own malware test host) | `[["sb.ssr",3,true,true,...]]` | `3` | Some pages unsafe (control) |
| `github.com` | `[["sb.ssr",4,false,false,false,...]]` | `4` | No site-level verdict (control) |
| **`ronit2405.github.io`** | `[["sb.ssr",3,false,false,true,false,false,...]]` | **3** | **Some pages on this site are unsafe** |
| `ronit2405.github.io/Day-matrix-tracker` | `[["sb.ssr",3,false,false,true,false,false,...]]` | **3** | Same host-level verdict |

Endpoint used:

```
https://transparencyreport.google.com/transparencyreport/api/v3/safebrowsing/status?site=ronit2405.github.io
```

Code reference: `1` = no unsafe content, `2` = this site is unsafe, `3` = **some pages on this site are unsafe**, `4` = no site-level verdict, `5` = hosts files not commonly downloaded, `6` = no data.

**What this proves:**

- Verdict is **identical for the bare host and for this project's path** → the finding is on `ronit2405.github.io`, not on `Day-matrix-tracker` specifically. All repos served from `<username>.github.io` share one host, so one flagged page downgrades every project on it.
- In the flags, **malware = false, unwanted software = false, social-engineering / deceptive content = true**. Google's classifier thinks something on this host *looks like a trick* — not that it serves malware.

> Google can change or clear a verdict only through a review. No edit to `index.html` can move this number. That is the whole point of this document.

---

## 2. This repo's audit — clean

What was checked and found (all files, 11 Nov 2026):

| Check | Result |
|---|---|
| External scripts, CDN, fonts, analytics, iframes, pixels | **None.** Zero outbound network references; the only `http(s)` strings in the page are its own `<link rel=canonical>` and the SVG namespace URI |
| `<form>`, password / email / OTP / payment / card / file inputs | **None** — every input is `text`, `date`, `checkbox`, `select`, `textarea` inside no form element |
| `eval`, `new Function`, `atob`, `String.fromCharCode`, obfuscated JS | **None** |
| Redirects (`location.href/assign/replace`), `window.open`, downloads, `Content-Disposition` tricks | **None** |
| `fetch` / `XMLHttpRequest` / `WebSocket` / `sendBeacon` / service worker | **None** |
| Permissions used | **None** — no geolocation, camera, microphone, clipboard *read*, notifications. Clipboard *write* only when the user presses "Copy" |
| Storage | `localStorage` key `dmt_v1` only. No cookies, no IndexedDB |
| CSP (enforced via `<meta http-equiv>`) | `default-src 'none'; style-src 'unsafe-inline'; script-src 'unsafe-inline'; img-src data:; base-uri 'none'; form-action 'none'` — the page **cannot** load or POST anything external |
| Backend | None. GitHub Pages serves static files only |

Runtime check (jsdom, headless) on all three tabs: set 48 slots, urgent/important flags, mood, rating, notes, add days, backup, restore, reset, theme cycle — **0 console errors, 0 network requests**.

**Conclusion: this project contains no phishing, malware, unwanted software or deceptive content.** Nothing here explains the warning, and nothing here can legitimately be "fixed" to remove it.

---

## 3. Actual likely cause: the shared host

Every page published on `ronit2405.github.io` inherits one reputation. Audited the other published repos:

| Repo | What's there | Risk signal |
|---|---|---|
| **`silent-sos-app`**, **`silent-sos-single-file`** (Pages: **ON**) | "Silent SOS — Emergency Alert Demo (Calculator UI)" — a **working calculator that hides an emergency-alert UI** behind a secret code (type `112` then `=`, or long-press `AC`) | ⚠️ **Strongest match for "deceptive / social engineering".** The page's own purpose is a disguised interface. It also requests **geolocation** and builds `wa.me` / `maps.google.com` links with the visitor's number and location. Whatever the intent, a disguised UI + location capture + third-party send links is exactly the pattern Google's classifier is built to catch. |
| `websolutions-studio` | Portfolio with a "Test Login Details / Login ID" block (Pages currently off) | ⚠️ Login-looking content on a host with no login |
| `Pooja-boutique-Management-` | Next.js app with **no `index.html`** at repo root → its Pages URL serves a "File not found" page | Noise, not a safety signal |
| `SGT`, `sgt-stock` | Firebase stock demo; `SGT/` has no root `index.html` (site is at `/SGT/sgt3/`), loads Firebase from `gstatic.com` with a **placeholder** config | Clean, but broken links |
| `myResumeDemo` (Pages: ON) | Resume with lorem ipsum | Clean |
| `Day-matrix-tracker` (this repo) | This app | **Clean** (section 2) |

**Google search has no idea which of your pages the flag is on.** The endpoint only says "some pages". The way to find the exact URL is Search Console's Security Issues report (section 5, Step B).

> **Separate real issue, unrelated to Safe Browsing:** `Pooja-boutique-Management-` has a committed `.env` file. Rotate whatever is in it and remove it from the repo/history regardless of this warning. (Checked: it is not reachable through Pages, since the Pages root has no app entry point.)

---

## 4. What was changed in this repo (and why it does *not* fix the warning)

While auditing, one genuine weakness was found **in this repo's own code** — not a phishing vector, but a self-inflicted one: the app writes user data into `innerHTML` and trusted the shape of the backup text / `localStorage` blob. A crafted "Restore from text" payload could inject markup, and a malformed-but-valid backup could break rendering (a `null` day name threw and left a blank screen).

Hardening applied to `index.html`:

- New `day()` / `norm()` sanitizers validate the whole state on **load** and on **restore**: unknown fields dropped, `u`/`i` limited to `Y`/`N`, slot indices limited to `0–47`, `rating` clamped to `0–10`, `theme` limited to `''`/`light`/`dark`, `date` must be `YYYY-MM-DD`, duplicate day ids de-duplicated, non-object days tolerated.
- Remaining interpolations escaped (`d.date`, day ids in option values and row `data-id`).
- Restore now re-renders immediately instead of leaving stale DOM.

Verified after the change: crafted payloads render **0 injected elements and 0 executed handlers**; corrupt JSON, wrong-shape blobs and legacy backups all still load cleanly; every existing feature (slots, bulk set, mood, rating, notes, multi-day, backup/restore, reset, theme) still passes with 0 console errors. Bonus: the in-page CSP already blocks `src`/`href` exfiltration, so even the old code could not have leaked data off-origin.

⚠️ **Be clear about this:** this hardening makes the app better and removes anything a reviewer could point at, but it **cannot** change the code-`3` verdict, because the verdict is on the host and the flags are deceptive-content flags, not code-quality flags.

---

## 5. How the warning actually gets removed

### Step A — Send the review request (the only action that can clear it)

Report endpoint: **https://safebrowsing.google.com/safebrowsing/report_error/?hl=en**

Use the **host root**, not just this path — that is where the verdict lives:

```
https://ronit2405.github.io/
```

Paste this (adjust the takedown sentence to what you actually do):

```
Hello Safe Browsing team,

Querying https://transparencyreport.google.com/transparencyreport/api/v3/safebrowsing/status?site=ronit2405.github.io
returns code 3 ("some pages on this site are unsafe") with the deceptive-content flag set
and the malware flag clear. The warning is therefore on the host ronit2405.github.io,
which is a GitHub Pages host shared by my portfolio demo projects.

I have audited every page published under this host. The page I need working is
https://ronit2405.github.io/Day-matrix-tracker/ - a single static HTML file, a personal
day planner. It has:
- no login, signup, password field, OTP, payment or card field, and no <form> at all
- no external scripts, CDN, fonts, analytics, trackers or iframes (zero network requests)
- no redirects, no downloads, no geolocation or other permission requests
- a strict CSP: default-src 'none'; style-src 'unsafe-inline'; script-src 'unsafe-inline';
  img-src data:; base-uri 'none'; form-action 'none'
- all data stored only in the visitor's own browser (localStorage key dmt_v1)

I have unpublished the pages under this host that could reasonably be read as deceptive,
in particular the two "Silent SOS" demo pages, which used a calculator interface to hide an
emergency-alert screen and asked for the visitor's location. They are no longer served.

Please re-review https://ronit2405.github.io/ and remove the warning. Thank you.
```

Send **one** request. Repeated submissions slow the review down.

### Step B — Find the exact flagged URL (Search Console)

This is the only way to learn *which* URL Google flagged, and its "Request review" route usually resolves faster than the anonymous report.

1. **https://search.google.com/search-console** → *Add property* → **URL prefix** → `https://ronit2405.github.io/Day-matrix-tracker/`. (A domain property for `github.io` is impossible — you do not control its DNS. A path prefix works.)
2. Verify with the **HTML file** method (Google gives you e.g. `google1a2b3c4d5e6f.html`; drop it in this repo's root and push — Pages re-serves it within a minute).
3. **Security & Manual actions → Security issues.** The flagged URL(s) are listed there.
4. Fix/unpublish that page, then press **Request review**.
5. **Indexing → Sitemaps** → submit `sitemap.xml` (already present, absolute URLs already correct). It helps Google re-crawl this site as a real, documented property.

### Step C — Remove the likely culprit from the host (do this *before* Step A)

- Repo → **Settings → Pages** → Source: **None** for `silent-sos-app` and `silent-sos-single-file`.
- Same for anything else with a disguised or login-like UI.
- If you want to re-publish the SOS demos later, put an unmissable banner *above* the calculator on first paint — a hidden UI is precisely what a deceptive-content classifier is trained to flag.
- For URLs you deleted: Search Console → **Removals → New request**.

### Step D — A working link while you wait

Review takes ~1–3 days, sometimes longer.

- **Netlify Drop** (no account): https://app.netlify.com/drop → drag this folder → instant `https://<random>.netlify.app`. `netlify.toml` in this repo already sets sensible security headers.
- Cloudflare Pages / Vercel also work: no build command, output directory `.`.
- `localStorage` is per-origin: take a backup on the old link (*Help & data → Show backup*) and *Restore from text* on the new one.

### Step E — Confirm it cleared

Re-query the endpoint until it returns `1`:

```
https://transparencyreport.google.com/transparencyreport/api/v3/safebrowsing/status?site=ronit2405.github.io
```

Chrome may keep showing a cached interstitial — retest in Incognito or after clearing cached files. Still `3` after your Step A/B request and 5+ days? Open a GitHub Support ticket with the repo link and a screenshot.

---

## 6. Separate problem found: the GitHub Pages URL currently 404s

While verifying the live site (11 Nov 2026), `https://ronit2405.github.io/Day-matrix-tracker/` returned, for `/`, `/index.html`, `/about.html`, `/robots.txt` and `/404.html` alike:

> **"There isn't a GitHub Pages site here."**

Facts:

- Pages settings read back healthy over the API: `status=built`, source `main` / `/`, `https_enforced=true`, custom 404 on.
- The build that ran after the last commit to `main` **succeeded** (build → report-build-status → deploy all green), and the deployment's `environment_url` is exactly that URL.
- The repository contents are intact at `main` (9 files at root), and the `github-pages` environment's only protection rule is a branch policy on `main`.
- The sibling project `myResumeDemo` (same host, same account, same Pages setup) **does** serve, so the host itself resolves — it is this project's origin that is not being served.
- GitHub status at check time: all systems operational.

So the failure is not in this repo's files: either the deployment has not been promoted/served at the edge for this path, or the Pages site needs to be re-saved. Try, in order:

1. **Settings → Pages** → re-select *Deploy from a branch* → `main` / `/` → **Save** (this re-publishes and often clears a stuck deployment).
2. Push any commit to `main` to trigger a fresh `pages build and deployment` run, then watch **Actions → pages-build-deployment** through the `deploy` job.
3. If it still 404s while `myResumeDemo` serves, open a **GitHub Support** ticket naming both URLs — that is a server-side Pages problem on their end, not something a code change can fix.

Note: with that URL currently down, a Safe Browsing reviewer has nothing to review. Restore the live URL **first**, then send the Section 5 request.

---

## 7. Honest expectations

| Action | Effect on warning |
|---|---|
| Editing `index.html` (any amount) | **None** — verdict is host-level and flag is deceptive-content |
| The hardening in section 4 | **None** for the warning; genuinely removes an injection/brittleness risk |
| Unpublishing the disguised "Silent SOS" pages | Removes the most likely trigger — required for a clean re-review |
| Safe Browsing report (Step A) + Search Console review (Step B) | **This is what clears it.** ~1–3 days after the trigger is gone |
| Moving to Netlify/Cloudflare | New hostname, no inherited verdict — immediate working link |

There is no button, header, `robots.txt` line or code change that can remove a Google Safe Browsing interstitial. Anyone who tells you otherwise is guessing.
