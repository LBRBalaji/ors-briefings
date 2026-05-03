# haanest.app — Claude Briefing Document
**Repo:** `LBRBalaji/ors-trace-origin` (private)  
**Live:** [haanest.app](https://haanest.app)  
**Previous URL:** land.orsone.app (retired — redirects to haanest.app)  
**Last updated:** 3 May 2026  
**PAT:** `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

> Read this before touching any file. Verify with `grep -n` before every edit.  
> Never assume — always confirm the exact string exists first.

---

## 1. What is haanest.app?

An integrated React/Vite web platform for **industrial land real estate transactions in India**. Built and operated by **Lakshmi Balaji ORS Private Limited**, Chennai. Owner: **Balaji Aram Valartha Sundaram** (non-developer — directs all development).

**Brand:** haanest — `haa` in Pine Green `#01796F`, `nest` in Brown Khaki `#8B7355`  
**Tagline:** Know Ground Reality — A **HONEST** Report  
**Favicon:** `/haanest-favicon.svg` — pine green rounded square with white `haa`

**Mission:** Take a land transaction from raw market sourcing through site evaluation, title analysis and seller verification — in one integrated platform.

**The four-step land transaction journey:**
1. **Source Land** — post demand, receive structured site submissions from market
2. **Site Verified** — threats, risks and challenges assessed (Evaluate + Evaluate-SVR)
3. **Title Verified** — ownership records, reconciliation and succession analysis
4. **Sellers Verified** — seller identity and authority confirmed

**Two stages, ten modules (including Evaluate-SVR):**

| Stage | Modules |
|-------|---------|
| Stage 1 — Sourcing & Evaluation | Deal Board · Tracker · Land Map · Evaluate · Evaluate-SVR |
| Stage 2 — Title Analysis | Ownership (ORS-1) · Reconciliation (ORS-2) · Succession (ORS-3) · Field Collect (ORS-FC) · Title Report (ORS-R) |

---

## 2. Stack

| Layer | Detail |
|-------|--------|
| Framework | React 18 + Vite 5 |
| Language | JavaScript (JSX) — no TypeScript anywhere |
| Styling | Inline styles only. One `global.css` for resets (legacy, not used by React) |
| Auth | Firebase Auth (email/password). Super admin: `balaji@lakshmibalajio2o.com` |
| Package manager | npm |
| Hosting | Vercel — auto-deploys on `git push origin main` (~60 seconds) |
| Repo | `LBRBalaji/ors-trace-origin` (private) |
| Remote URL | `https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git` |
| Routing | Manual state management — `src/shared/router.js` + `App.jsx`. No React Router. |

**Dependencies** (package.json — no others):
```json
"dependencies": {
  "firebase": "^10.12.2",
  "react": "^18.3.1",
  "react-dom": "^18.3.1"
}
```

---

## 3. Brand Assets (May 2026)

Three PNG files in `/public/` — use the correct one per background:

| File | Background | Text colour | Use on |
|------|-----------|-------------|--------|
| `haanest-wordmark.png` | White (original, padded) | Green + khaki | — (do not use directly, use -trans) |
| `haanest-wordmark-trans.png` | Transparent | Green + khaki | White/light backgrounds (HomePage nav) |
| `haanest-wordmark-white.png` | Transparent | Pure white | Dark green headers (all module headers) |
| `haanest-favicon.svg` | Pine green rounded square | White `haa` | Browser tab favicon |

**CRITICAL:** The original `haanest-wordmark.png` has 503px white padding top and 561px bottom (total 1536px tall, text only 472px). It was cropped to `haanest-wordmark-trans.png` (2408×512, transparent bg). Using the original PNG with `filter: brightness(0) invert(1)` produces a white rectangle — always use the `-trans` or `-white` variants.

**Standard nav implementation (dark green header):**
```jsx
<img src="/haanest-wordmark-white.png" alt="haanest"
  style={{ height: '26px', width: 'auto', display: 'block' }} />
```

**Standard nav implementation (white header):**
```jsx
<img src="/haanest-wordmark-trans.png" alt="haanest"
  style={{ height: '26px', width: 'auto', display: 'block' }} />
```

---

## 4. Colour System (May 2026)

**haanest brand colours — applies to all modules:**

```
Pine Green:  #01796F  (NAVY constant — primary buttons, headers, active states)
Brown Khaki: #8B7355  (BLUE constant — accent, links, secondary highlights)
White bg:    #fafaf8
Muted text:  #536471
Border:      #e8e8e4
```

**Historical note:** Before May 2026, this platform used `#1a3a5c` (navy) and `#1d9bf0` (blue). These are now fully replaced with `#01796F` and `#8B7355` across ALL modules. The old `global.css` retains these for legacy reasons only — it is not used by any React component.

**Three strict theme groups — NEVER cross-apply:**

**Group 1 — Deal Board · Tracker · Land Map** (own CSS-in-JS via `styles.js`)
```
CSS vars: --blue: #1d9bf0 (retained internally for DealBoard/LandMap — do NOT change)
          --navy: #1a3a5c (retained internally — do NOT change)
Fonts: DM Sans (body), Space Mono (labels)
```
⚠️ DealBoard/Tracker/LandMap have their own `styles.js` CSS injection. Do NOT apply haanest green/khaki to these modules — they follow their own internal design system.

**Group 2 — ORS-1/2/3/FC/R · Admin · AppShell**
```
NAVY:   #01796F  (Pine Green — header bg, primary buttons)
BLUE:   #8B7355  (Khaki — accents)
Font:   Helvetica Neue / Arial
```

**Group 3 — Evaluate · Evaluate-SVR**
```
Header bg: #01796F (Pine Green, same as Group 2)
CSS vars: --navy: #01796F, --blue: #8B7355 (updated in styles.js)
Fonts: Arial (not DM Sans — Evaluate has its own EVAL_CSS)
```

**AppShell constants** (`src/components/AppShell.jsx`):
```js
const NAVY   = '#01796F'  // Pine green
const PURPLE = '#8B7355'  // Khaki (named PURPLE historically)
const PURPLEM= 'rgba(139,115,85,0.65)'
```

---

## 5. Routing & Navigation (May 2026)

### URL-based routing — `src/shared/router.js`

No React Router library. Routing is handled by three mechanisms working together:

1. **`src/shared/router.js`** — maps clean URL paths to internal screen IDs
2. **`App.jsx` useEffect (mount)** — reads `window.location.pathname` on load, sets initial screen
3. **`App.jsx` useEffect (screen change)** — writes pathname to URL via `history.replaceState` on every screen change
4. **`popstate` listener** — syncs screen when browser Back/Forward buttons are pressed
5. **Vercel catch-all** — `vercel.json` rewrites all paths to `index.html`

**Route table:**

| URL Path | Screen ID | App | Document Title |
|----------|-----------|-----|----------------|
| `/` | `null` | LandingPage (post-login) / HomePage (public) | haanest — Industrial Land Transactions |
| `/dealboard` | `DB-APP` | Deal Board | Deal Board — haanest |
| `/evaluate` | `EVAL` | Evaluate | Site Evaluate — haanest |
| `/ownership` | `ORS-1` | Ownership | Ownership — haanest |
| `/reconciliation` | `ORS-2` | Reconciliation | Reconciliation — haanest |
| `/succession` | `ORS-3` | Succession | Succession — haanest |
| `/fieldconnect` | `ORS-FC` | Field Connect | Field Connect — haanest |
| `/report` | `ORS-R` | Title Report | Title Report — haanest |
| `/admin` | `ADMIN` | Admin | Admin — haanest |
| `/svr` | `SVR` | Evaluate-SVR | Evaluate-SVR — haanest |
| `/<unknown>` | `'404'` | 404 screen | Not Found — haanest |

**`src/shared/appUrls.js`** — derives `APP_URLS` from `router.js`:
```js
export const APP_URLS = Object.fromEntries(
  ROUTES.map(r => [r.id, `${BASE}${r.path}`])
)
// e.g. APP_URLS['ORS-1'] === 'https://haanest.app/ownership'
```
LandingPage app tiles call `window.open(APP_URLS[id], '_blank')` — opens app in new tab.  
Hamburger menu calls `navigate(id)` → `setScreen(id)` — switches within current tab.

**App.jsx navigate function:**
```js
function navigate(id) {
  if (!id || id === 'CONTACT') { goContact(); return }
  setProductId(null)
  if (id.startsWith('TA-')) { setProductId(id); return }  // product info page
  const path = screenToPath(id)
  if (path) pushPath(path)   // push to browser history
  setScreen(id)
}

function goHome() {
  replacePath('/')
  setScreen(null)
}
```

**404 screen:** Rendered inline when `pathToScreen()` returns `'404'`. Shows haanest wordmark, large 404, the invalid path, and a "← Back to Home" button.

---

## 6. Three Firebase Projects

**CRITICAL — Three separate Firestore databases. Getting this wrong causes silent data failures.**

### Project 1: `ors-system-3480b` — Main platform
**Used by:** ORS-1, ORS-2, ORS-3, ORS-FC, ORS-R, Admin, Universal Search, SVR user access checks  
**Config:** `src/firebase.js`  
**Export:** `db`

**Collections:**
| Collection | Contents |
|------------|----------|
| `ors-users/{uid}` | All user profiles, permissions, validity |
| `sites/{siteId}` | All Title Analysis site data |

**User profile fields (May 2026 additions):**
```js
eval_access:       boolean,   // access to Evaluate module
svr_access:        boolean,   // access to Evaluate-SVR (field verification)
platform_manager:  boolean,   // admin-level access within Evaluate (not full superadmin)
db_role:           string,    // Deal Board role
ors_apps:          string[],  // Title Analysis apps
ors_projects:      string[],  // site IDs (empty = all)
```

### Project 2: `ors-demand-platform` — Transaction Pipeline
**Used by:** Deal Board · Tracker · Land Map  
**Config:** `src/modules/DealBoard/firebase.js`  
**Key:** `AIzaSyAssF4cjQCOuY-Ww6mEBIUMd78rJH8g_gY`

⚠️ DealBoard/Tracker/LandMap have their own maps API key: `AIzaSyCON67DgHi7fBa0C3TpbEPuAB4FwhsnIY8`  
This key must have both `haanest.app` AND `www.haanest.app` in HTTP referrer restrictions.

### Project 3: `evaluate-6f1bf` — Evaluate + Evaluate-SVR
**Used by:** Evaluate module AND Evaluate-SVR (same tab, same module)  
**Config:** `src/modules/Evaluate/firebase.js`  
**Key:** `AIzaSyDChmWPMBUn_SCqfdfXFT_zinmspmZA0vs`  
**Export:** `dbEval` (NOT `db` — NEVER import `db` from `../../firebase` for Evaluate data)

**Collections:**
| Collection | Contents |
|------------|----------|
| `land_reports` | All Evaluate reports. Also stores SVR findings + custom field definitions on same document |

**land_reports document schema (May 2026):**
```js
{
  // Core site identification (Section 01)
  siteId, siteCoordinate, locationCircle, landSize,
  district1, taluk1, village1, district2, taluk2, village2,

  // Sections 02-05: accessibility, infra distances, digital assets

  // Section 06 — Site Verification Findings
  officialCategory, siteFrontage, siteLevel,
  encroachment, htTowerLine, htTowerBase, waterChannelRevenue,
  waterBodyRevenue, waterBodyOnsite, govtLand, landAcquisition,
  landCeiling, landReforms, quarry, burial, asi, forest,

  // Section 07 — Compliance Findings
  landUseConversion, panChamiLand, conversionExtent, pwdPermission,
  courtOrder, swappingAnandheenam, swappingTemple, sipcotNoc,
  highwaysNoc, asiNoc, ongcNoc, acquisitionNoc, otherConditions,

  // Custom fields (admin-defined per site, before SVR user opens)
  customFieldsS06: [{id, label, type, options}],  // type: 'yesno'|'reqd'|'text'
  customFieldsS07: [{id, label, type, options}],

  // Custom field responses (by field id, stored on same doc)
  // format: form[fieldId] for toggle, form[fieldId+'_note'] for text note

  // Report By
  siteVerificationBy, siteVerificationContact,
  complianceVerificationBy, complianceVerificationContact,

  // Section 08
  additionalInfo,

  // SVR (Evaluate-SVR field app writes these)
  assignedSvrEmail, assignedSvrUid, assignedAt,
  svrGeoCapture,      // "lat, lng" string captured on device
  svrGeoTime,         // ISO timestamp of geo capture
  svrEncroachment, svrHtTowerLine, svrHtTowerBase, svrWaterChannel,
  svrWaterBody, svrWaterBodyOnsite, svrGovtLand, svrLandAcquisition,
  svrLandCeiling, svrLandReforms, svrQuarry, svrBurial, svrAsi,
  svrForest, svrPanChamiLand, svrOfficialCategory, svrSiteFrontage,
  svrSiteLevel, svrNotes, svrCompletedBy, svrCompletedAt,
  // Custom SVR field responses: svr_cf_{fieldId}, svr_cf_{fieldId}_note
  svrUpdatedAt,       // serverTimestamp() on each SVR save

  // Timestamps
  createdAt, updatedAt,
}
```

---

## 7. Module Inventory (May 2026)

### `src/shared/router.js` ← NEW
Path ↔ screen ID mapping. Exports: `ROUTES`, `pathToScreen()`, `screenToPath()`, `screenToTitle()`, `pushPath()`, `replacePath()`. Used by `App.jsx` and `appUrls.js`.

### `src/shared/appUrls.js` ← UPDATED
Derives `APP_URLS` map from `router.js`. Used by `LandingPage.jsx` for new-window tile opens. Replaces the old `export const APP_URLS` that was in `App.jsx` (circular import fix).

### `src/pages/HomePage.jsx` ← UPDATED
Public marketing page. Full haanest brand theme (pine green `#01796F`, khaki `#8B7355`). Nav: `haanest-wordmark-trans.png` at 26px + BETA badge + tagline. CTA section uses pine green background.

### `src/pages/LandingPage.jsx` ← UPDATED
Post-login personalised dashboard. Nav: `haanest-wordmark-white.png` at 26px + BETA badge + tagline. App tiles open in **new window** via `window.open(APP_URLS[id], '_blank')`. Includes Evaluate-SVR card.

### `src/components/AppShell.jsx` ← UPDATED
Universal header. Brand: `haanest-wordmark-white.png` at 22px. Colors: NAVY=`#01796F`, PURPLE=`#8B7355`. Footer gradient updated to green/khaki. Module badge and tab underlines use white tints on dark header.

### `src/modules/Evaluate/` ← MAJOR UPDATE (May 2026)

**`index.jsx`** — 1899 lines (was 784). Components inside the file:

| Component | Lines | Purpose |
|-----------|-------|---------|
| `EvalHeader` | ~100 | Single unified pine green header — replaces old dual AppShell+`<header>` stack |
| `UserDashboard` | ~80 | Overlay showing user's own reports with stats |
| `CustomFieldEditor` | ~120 | Admin UI to add/remove/reorder custom fields per section |
| `CustomFieldDisplay` | ~60 | Renders custom field inputs (toggle + notes) in InputView |
| `SVRCustomField` | ~60 | Renders one custom field in SVRView (toggle + notes) |
| `NavTab` | ~20 | Tab button with active underline |
| `SVRView` | ~300 | Evaluate-SVR field app (site list → field findings form) |
| `AdminView` | ~200 | User management + site assignment for Evaluate |
| `ReportsView` | ~200 | Saved reports dashboard |
| `InputView` | ~200 | 8-section input form |
| `PreviewView` | ~150 | Print preview |

**Header architecture:** Single `EvalHeader` component. Removed the old dual-header (AppShell stacked on Evaluate `<header>`). The Evaluate module no longer uses `AppShell` at all.

**Views (role-based tabs):**

| View | Who sees it |
|------|-------------|
| `reports` | Admin · Eval users |
| `input` / `preview` | Admin · Eval users |
| `svr` | Admin · SVR users (auto-routed on login if svr-only) |
| `admin` | Admin (superadmin or platform_manager) only |

**Permission flags (derived in main component):**
```js
const isAdmin    = orsProfile?.role === 'superadmin' || !!orsProfile?.platform_manager
const isSvrUser  = !!orsProfile?.svr_access && !isAdmin
const isEvalUser = !!orsProfile?.eval_access || isAdmin
```

**EvalHeader features:**
- `haanest-wordmark-white.png` at 26px on pine green `#01796F` background
- BETA badge
- Tagline: "Know Ground Reality — A **HONEST** Report"
- Sign Out button (visible when logged in)
- Hamburger menu → My Dashboard overlay, ← haanest Home, Sign Out
- No user email displayed, no "ORS Home" button

**Scroll fix:** `ReportsView` outer div: `paddingTop: '60px'`. Input/Preview app-body: `paddingTop: '80px'`. Nav strip: `position: sticky; top: 60px; zIndex: 900`.

**Custom fields (per-site, admin-defined):**
- Admin opens a report → Input Data → S06 or S07 → clicks "+ Add custom field to this section"
- Provides: label text + answer type (Yes/No/Not Sure · Required/Not Required/Maybe · Text only)
- Fields stored as `customFieldsS06: [{id, label, type, options}]` on the report document
- Custom fields appear in: InputView (editable), Preview (read-only), PDF download, SVRView (toggle + notes)
- Custom field answers stored directly on the report doc as `form[fieldId]` + `form[fieldId+'_note']`
- SVR responses stored as `svr_cf_{fieldId}` + `svr_cf_{fieldId}_note`

**Section names (May 2026):**
- Section 06: `SITE VERIFICATION FINDINGS` (was "Key Due Diligence Findings")
- Section 07: `COMPLIANCE FINDINGS` (was "Conditions")
- New field in S07: `panChamiLand` — "Panchami (DC) Land Within Boundary" (Yes/No/Not Sure)
- New fields at S07 bottom: "Report By" — site verification name+contact, compliance name+contact

**PDF download:**
- Button: "⬇ Download PDF" in Preview toolbar
- Built from `form` state (not DOM)  
- Opens print-ready HTML in new window — user prints → Save as PDF
- Includes: pine green header band with wordmark, all sections, custom fields, Report Generated By block, Lakshmi Balaji ORS footer

### `src/modules/Evaluate/styles.js` ← UPDATED
CSS vars: `--navy: #01796F`, `--blue: #8B7355`. Header background: `#01796F`. Logo-block img: `height: 26px`. Mobile responsive rules added (`@media (max-width: 768px)` and `480px`).

### Evaluate-SVR (`view === 'svr'` within Evaluate module)

Accessed at **`haanest.app/svr`** → routes to `Evaluate` module with `initialView='svr'`.

**SVRView — field user experience:**
1. Opens to list of assigned sites (admin sees all; SVR user sees only their assigned sites)
2. Click a site → detail view with:
   - Auto-populated site info (read-only, from Evaluate S01 data)
   - **📍 Capture Current Location** — browser Geolocation API, stores `svrGeoCapture` + `svrGeoTime`
   - S06 Site Verification Findings — all standard fields with Yes/No/Not Sure toggles
   - Custom fields from S06 — auto-synced from Firestore via `onSnapshot`, each with toggle + notes textarea
   - S07 Compliance custom fields (if any) — same pattern
   - Field Notes textarea
   - Save → writes `svr_*` fields to same `land_reports` Firestore document
3. SVR user can save multiple times (updates overwrite). Admin sees SVR progress in real-time.

**Real-time sync:** `subscribeReport(reportId, cb)` uses Firestore `onSnapshot` — SVR changes appear in Evaluate's reports view instantly without refresh.

**AdminView — site assignment + user access:**
- Tab 1: Site Assignment — per report, assign an SVR user from dropdown (filters to svr_access or eval_access users)
- Tab 2: User Access — toggle `eval_access`, `svr_access`, `platform_manager` per user with one click
- Writes access flags to `ors-users/{uid}` on `ors-system-3480b`

**LandingPage card:** "Evaluate-SVR · Field Verification App" — opens `haanest.app/svr`

---

## 8. User Permission System (May 2026)

### `ors-users/{uid}` new fields
```js
svr_access:        boolean,   // can access Evaluate-SVR field app
platform_manager:  boolean,   // admin-level in Evaluate (site assignment, user management)
```

### Access rules (full table)

| Module | Requires |
|--------|---------|
| `EVAL` | `eval_access === true` OR superadmin |
| `SVR` | `svr_access === true` OR `eval_access === true` OR superadmin |
| `ADMIN (Evaluate)` | `platform_manager === true` OR superadmin |
| `DB` / `TR` | `db_role` any non-empty |
| `MAP` | `db_role === 'admin'` OR `'agent'` |
| `ORS-1/2/3/FC/R` | `ors_apps` includes that ID |
| `ADMIN` | `role === 'admin'` OR superadmin |

SVR-only users (`svr_access: true`, no `eval_access`, not superadmin): auto-routed to `svr` view on login. They never see Reports, Input, or Admin tabs.

---

## 9. Key Technical Patterns

### Python3 for multi-line JSX replacements
`sed` fails on multi-line blocks. Always use Python3:
```bash
python3 << 'PYEOF'
with open('src/path/to/file.jsx', 'r', encoding='utf-8') as f:
    c = f.read()
old = """exact multi-line string"""
new = """replacement"""
if old in c:
    c = c.replace(old, new)
    print("OK")
else:
    print("MISS — string not found, check exact whitespace")
with open('src/path/to/file.jsx', 'w', encoding='utf-8') as f:
    f.write(c)
PYEOF
```

### Always verify before editing
```bash
grep -n "exact string" src/path/to/file.jsx
# 0 results = string not there, do NOT proceed
# 1 result = safe to replace
# 2+ results = identify which instance
```

### Byte-level editing for styles.js
`styles.js` stores a CSS string in a JS template literal with escaped `\n` sequences. Use byte-level replacement:
```bash
python3 << 'PYEOF'
with open('src/modules/Evaluate/styles.js', 'rb') as f:
    raw = f.read()
# Literal \n in raw file is b'\\n'
old = b'old css\\n        rule here'
new = b'new css\\n        rule here'
if old in raw:
    raw = raw.replace(old, new)
    print("OK")
with open('src/modules/Evaluate/styles.js', 'wb') as f:
    f.write(raw)
PYEOF
```

### Always build before pushing
```bash
npm run build
# Must output: ✓ built in X.XXs (no errors)
# Pre-existing warnings (chunk size, duplicate key in ui.jsx, dynamic import) are normal — ignore
```

### Git workflow
```bash
git config user.email "balaji@lakshmibalajio2o.com"
git config user.name "Balaji"
git add -A
git commit -m "fix/feat: description"
git push origin main
# Vercel deploys in ~60 seconds
```

### Standard clone (fresh container)
```bash
git clone https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git /home/claude/ors-platform
cd /home/claude/ors-platform
npm install
```

---

## 10. DealBoard Scroll Fix (unchanged — critical)

Fixed layers: AppShell 52px + DealBoard header 60px + floating-search 46px = **158px total**.
```css
.demands-layout  { padding-top: 160px; }         /* 2px breathing room */
.sidebar         { position: sticky; top: 156px; height: calc(100vh - 156px); }
.sidebar-right   { position: sticky; top: 156px; height: calc(100vh - 156px); }
```
Do NOT add `height: calc(100vh - Npx); overflow: hidden` to `.app-body`.

---

## 11. Known Issues & Warnings

### LandMap — Google Maps API referrer error
Maps API key `AIzaSyCON67DgHi7fBa0C3TpbEPuAB4FwhsnIY8` must have all four of these in HTTP referrer restrictions in Google Cloud Console:
```
haanest.app
haanest.app/*
www.haanest.app
www.haanest.app/*
```
The `www.haanest.app` variant is required — the error message says: "Your site URL to be authorized: https://www.haanest.app/"

### Evaluate — circular import (solved)
`APP_URLS` was briefly in `App.jsx` causing `LandingPage → App → LandingPage` circular import. Fixed by moving to `src/shared/appUrls.js`. Do not move `APP_URLS` back into `App.jsx`.

### Admin personas.js — TDZ crash
`PERSONA_META` **must** stay in `src/modules/Admin/personas.js`. Never move it back into `Admin/index.jsx`.

### Evaluate — dual header (solved)
Old architecture had AppShell (52px) stacked on a separate Evaluate `<header>` (60px), causing 112px of fixed chrome and content hidden beneath. Fixed: Evaluate now uses its own single `EvalHeader` component — no AppShell in Evaluate.

### Pre-existing build warnings (safe to ignore)
```
- Some chunks are larger than 500 kB (no action needed)
- Duplicate key "lineHeight" in src/shared/ui.jsx (pre-existing, not breaking)
- Dynamic import of Evaluate/firebase.js (pre-existing)
```

### Colour audit command
```bash
grep -rn "#1a3a5c\|#1d9bf0\|#6141ac\|97,65,172\|hsl(259" src/ --include="*.jsx" --include="*.js" | grep -v "global.css\|DealBoard/styles.js\|LandMap\|Tracker"
# DealBoard/styles.js and LandMap CSS intentionally retain #1a3a5c/#1d9bf0
# global.css intentionally retains old values — not used by React
```

---

## 12. Completed Features Changelog

### May 2026 — haanest brand + routing + Evaluate overhaul

**Brand & Identity**
- Platform renamed from `land.orsone.app` → **`haanest.app`**
- Brand colours: Pine Green `#01796F` + Brown Khaki `#8B7355`
- Three wordmark assets: original PNG, transparent-bg, white-text
- SVG favicon: pine green square + white `haa`
- Tagline everywhere: "Know Ground Reality — A **HONEST** Report"
- BETA badge in all nav headers

**Consistent header across all modules:**
- `HomePage` nav: white bg, `haanest-wordmark-trans.png` (coloured)
- `LandingPage` nav: pine green bg, `haanest-wordmark-white.png`
- `AppShell AppHeader` (ORS-1/2/3/FC/R, Admin): pine green bg, white wordmark
- `Evaluate EvalHeader`: pine green bg, white wordmark (replaced dual-header)
- `DealBoard DBHeader`: pine green bg, white wordmark
- `LandMap` header: pine green bg, white wordmark
- `DealBoard UserAuth` modal: pine green bg, white wordmark

**URL routing (clean paths):**
- `src/shared/router.js` created — central path ↔ screen map
- `src/shared/appUrls.js` updated — derives URLs from router
- All 9 apps have clean paths: `/dealboard`, `/evaluate`, `/ownership`, etc.
- `/svr` path → Evaluate-SVR
- Browser Back/Forward work correctly
- Refresh at any URL restores the correct app
- Document `<title>` updates per app
- 404 screen for unknown paths
- LandingPage tiles open apps in new window (`window.open(_blank)`)
- Hamburger menu switches in current window

**Evaluate module — complete overhaul:**
- Section 06 renamed: "Site Verification Findings"
- Section 07 renamed: "Compliance Findings"
- New field S07: Panchami (DC) Land Within Boundary
- Report By fields: 4 name + contact fields at S07 bottom
- PDF download: branded print-ready HTML, all sections, custom fields, Report By
- Single `EvalHeader`: pine green, white wordmark, BETA, tagline, sign-in/out, hamburger
- Removed: dual-header, user email display, ORS Home button
- User dashboard overlay (hamburger → My Dashboard)
- Scroll fix: content no longer hidden under fixed header
- Mobile responsive CSS added to `EVAL_CSS`

**Custom fields (per-site, admin-defined):**
- Admin adds fields to S06 or S07 before assigning to SVR user
- Three answer types: Yes/No/Not Sure, Required/Not Required/Maybe, Text only
- Toggle + free text notes per field
- Custom fields appear in: InputView, Preview, PDF, SVRView
- Stored as `customFieldsS06/S07: [{id, label, type, options}]` on `land_reports` doc
- SVR responses stored as `svr_cf_{id}` + `svr_cf_{id}_note`

**Evaluate-SVR (new app, view within Evaluate):**
- Accessible at `haanest.app/svr`
- Role: `svr_access: true` on `ors-users` profile
- SVR-only users auto-routed to SVR tab on login
- Site list filtered to assigned sites (admin sees all)
- Geo capture via browser Geolocation API (lat/lng + timestamp)
- S06 standard + custom fields: toggle + notes per field
- S07 custom fields (if defined by admin)
- Real-time sync via Firestore `onSnapshot` — Evaluate sees SVR updates instantly
- Save writes `svr_*` fields to same `land_reports` document

**Evaluate Admin view (tab within Evaluate):**
- Site Assignment tab: assign SVR user per report from dropdown
- User Access tab: toggle `eval_access`, `svr_access`, `platform_manager` per user
- Writes to `ors-users/{uid}` on `ors-system-3480b`
- Accessible to superadmin or `platform_manager: true` users

**LandingPage:**
- Evaluate-SVR card added (Stage 1, Brown Khaki accent)
- All cards open apps in new window

---

## 13. Pending / Not Started

- [ ] LandMap — refresh after adding `www.haanest.app` to Maps API key referrers
- [ ] ORS-FC Field Collect — minimal stub only (32 lines) — full mobile form not built
- [ ] Predictive Analytics — marked "Coming Soon" (Gemini 1.5-flash deprecated)
- [ ] AI description generation — marked "Coming Soon"
- [ ] Evaluate-SVR — photo upload per field (currently toggle + text only)
- [ ] Evaluate-SVR — offline/PWA support (currently requires network)

---

## 14. Real User Accounts

| Name | Email | Role |
|------|-------|------|
| Balaji | `balaji@lakshmibalajio2o.com` | superadmin (hardcoded) |
| Ejaz Nathani | `ejaz_nathani@welspun.com` | Welspun — real developer account |
| Raj Attri | `raj.attri@ccigroup.co.in` | CCI — real developer account |

---

## 15. 3 May 2026 — Changes & Fixes (today's session)

### Homepage (`src/pages/HomePage.jsx`) — restructured

**Section order fixed:**
Evaluate → Output (moved here, part of Evaluate) → Trust Bar → Title Analysis → Land Sourcing

**New sections added:**
- **Trust Bar** (pine green) — "Half Acre or Hundred Acres. Be Sure of Your Investment. / Scale doesn't change the Risk. Get Evaluated: Risks | Challenges | Opportunities."
- **Title Analysis description** — three cards (Ownership / Reconciliation / Succession, each with Record Input / Gap Analysis / Heir Analysis labels) + Report strip (4 bullet points: Transaction Scorecard, Risk Flags, Mitigation Roadmap, Site & Seller Audit)
- **Land Sourcing** rebuilt — "Green Field" in pine green + "Brown Field" in khaki, new hero title ("Like it or Not, Real Estate is a Minefield."), 5-row stage table (Stage | The Pain | The HAAnest Solution)

**Colour fix:** All `#0f1419` (black) backgrounds replaced with `#014a45` (dark teal) or `#01796F` (pine green). No black backgrounds remain.

**Mobile responsive:** All grids collapse ≤768px. `hp-hero-sect` gets `padding: 56px 20px 48px` on mobile.

**"Evaluate Your Land" highlight:** Rendered at 20px, font-weight 900, full white — stands out from surrounding tagline text.

**All CTA buttons:** `borderRadius: 0` (flat, no rounding) throughout.

### AppShell (`src/components/AppShell.jsx`)

- **Breadcrumb removed:** Home / DB-APP / module-name links removed from AppShell header left side. Navigation entirely via hamburger.
- **Tagline — single line only:** Second line "Verification Report: Site | Title | Seller" removed from all header brand blocks. Only `Know Ground Reality — A HONEST Report` remains in headers.
- **All module headers WHITE:** Background changed from dark green `#01796F` to `#fff`. Wordmark uses `haanest-wordmark-trans.png` (coloured text on transparent).

### Deal Board search moved to AppShell header

- Floating `#floating-search` div removed from `DemandsListView.jsx`
- State `dbSearch` lifted to `DealBoard/index.jsx`
- Compact flat search input rendered in AppShell `actions` prop
- Passed as `externalSearch` prop to `DemandsListView`
- Layout offsets corrected: `padding-top: 116px` (was 162px — floating search was 46px of that)
- Sidebars: `top: 112px; height: calc(100vh - 112px)`

### Tracker — Evaluate button restored

`openEvaluate(s)` in `SiteCard.jsx` — mirrors original `tracker.html` logic. Opens `/evaluate?siteId=...&coord=...&village=...&acres=...` in new tab. Evaluate reads these URL params on mount and prefills S01 fields, then opens Input view directly.

### Evaluate — prefill from URL params (new)

On mount, reads `siteId`, `coord`, `village`, `landmark`, `acres`, `district`, `taluk` from URL query string. Maps to form fields and opens Input view. URL cleaned after reading (no re-prefill on refresh).

### LandMap — timeout and clear error (code fix)

`loadGoogleMaps()` now has an 8-second timeout. If the `_mapReadyLM` callback doesn't fire (referrer blocked), rejects with a clear message. Previously hung silently forever showing "Loading map…".

**Root cause:** Maps API key `AIzaSyCON67DgHi7fBa0C3TpbEPuAB4FwhsnIY8` missing `www.haanest.app/*` in HTTP referrers. **Balaji must add in Google Cloud Console:**
```
haanest.app   haanest.app/*   www.haanest.app   www.haanest.app/*
```

### Field Connect (`ORSFC`)

Rebuilt as full-screen iframe serving `/public/fieldconnect.html`. The original 1,644-line working HTML app is used as-is — no React rewrite. haanest theme applied directly to the HTML (pine green colours, wordmark in login + header, flat buttons). React wrapper adds AppHeader with hamburger.

### Title Analysis description (new homepage section)

Three-column cards: Ownership (Record Input / Chain of Custody), Reconciliation (Gap Analysis / 12+ Automated Checks), Succession (Heir Analysis / Total Genealogical Clarity). Report strip below (pine green, 4 bullet points with exact copy from brief). Mobile: cards stack to single column.

### Evaluate Admin — User Management rebuilt

Complete rebuild of `AdminView`. Features:
- Add User form (email, name, role → creates `ors-users` doc on `ors-system-3480b`)
- Table: User/Email · Status (ACTIVE/ON HOLD/FROZEN) · Role · Evaluate · SVR Field · Platform Mgr · Actions
- ACTIVATE / HOLD / FREEZE status buttons
- One-click privilege toggle chips
- Search filter
- Superadmin rows protected
- Uses `dbMain` (static import) — NOT dynamic import (dynamic was the bug)

### Evaluate — auth fixes (root cause resolved)

- `authEval` now correctly imported from `./firebase` (evaluate-6f1bf own auth)
- Auth listener race condition fixed — two useEffects work in sequence
- `handleLogin` signs into both Firebase projects simultaneously
- Firestore rules for evaluate-6f1bf: `allow read, write: if request.auth != null`

### Evaluate-SVR access guard fix

`canAccess('SVR')` in `useAuth.js` has no SVR rule — always returns false. App.jsx now checks `profile?.svr_access || profile?.eval_access || profile?.platform_manager` directly for SVR screen access.

---

*Update this file at the end of every session. Push to `LBRBalaji/ors-briefings`, file `LAND-BRIEFING.md`.*
