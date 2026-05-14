# haanest.app — Claude Briefing Document
**Repo:** `LBRBalaji/ors-trace-origin` (private)
**Live:** [haanest.app](https://haanest.app)
**Last updated:** 3 May 2026
**PAT:** `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` ← redacted. Balaji provides real token each session.

> READ THIS ENTIRE FILE BEFORE TOUCHING ANY CODE.
> Verify every string with grep -n before editing. Never assume. State root cause before any fix.

---

## 1. What is haanest.app?

An integrated React/Vite web platform for **industrial land real estate transactions in India**.
Built by **Lakshmi Balaji ORS Private Limited**, Chennai.
Owner: **Balaji Aram Valartha Sundaram** (non-developer — directs all development).

**Brand:**
- Name: haanest — `haa` in Pine Green `#01796F`, `nest` in Brown Khaki `#8B7355`
- Header tagline: `Know Ground Reality — A HONEST Report` (HONEST bold)
- Favicon: `/haanest-favicon.svg` — pine green square with white `haa`
- Footer: `Lakshmi Balaji ORS Private Limited · Building Transaction Ready Assets`

**Ten modules across two stages:**

| Stage | Modules |
|-------|---------|
| Stage 1 — Sourcing & Evaluation | Deal Board · Tracker · Land Map · Evaluate · Evaluate-SVR |
| Stage 2 — Title Analysis | Ownership (ORS-1) · Reconciliation (ORS-2) · Succession (ORS-3) · Field Connect (ORS-FC) · Title Report (ORS-R) |

---

## 2. Stack

| Layer | Detail |
|-------|--------|
| Framework | React 18 + Vite 5 |
| Language | JSX only — no TypeScript |
| Styling | Inline styles. Per-module CSS injection. `global.css` is legacy resets — not used by React |
| Routing | Manual state in `App.jsx` + `src/shared/router.js` — no React Router library |
| Hosting | Vercel — auto-deploys on `git push origin main` (~60 seconds) |
| Repo | `LBRBalaji/ors-trace-origin` (private) |
| Remote | `https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git` |

**Dependencies (nothing else):**
```json
{ "firebase": "^10.12.2", "react": "^18.3.1", "react-dom": "^18.3.1" }
```

---

## 3. Colour System

```
Pine Green:  #01796F   ← NAVY constant everywhere
Brown Khaki: #8B7355   ← BLUE constant everywhere
White bg:    #fafaf8
Border:      #e8e8e4
Muted text:  #536471
Body text:   #0f1419
```

**All module headers are WHITE background** (`#fff`, `border-bottom: 1px solid #e8e8e4`) with the coloured wordmark.

**Three wordmark PNG assets in `/public/`:**

| File | Background | Text colour | Use on |
|------|-----------|-------------|--------|
| `haanest-wordmark-trans.png` | Transparent | Green + khaki | All white/light headers |
| `haanest-wordmark-white.png` | Transparent | Pure white | (available but headers are now white — use -trans) |
| `haanest-favicon.svg` | Pine green square | White `haa` | Browser tab favicon |

**CRITICAL:** Original `haanest-wordmark.png` has 503px white padding top+bottom (1536px tall, text only 472px). Always use `-trans` or `-white` variants — never the original.

**AppShell constants:**
```js
const NAVY   = '#01796F'   // Pine green
const PURPLE = '#8B7355'   // Khaki (named PURPLE historically)
```

**DealBoard/Tracker/LandMap** retain internal `--navy: #1a3a5c` / `--blue: #1d9bf0` in their `styles.js`. Do NOT change these — they are independent design systems.

---

## 4. Three Firebase Projects — CRITICAL

Getting this wrong causes silent data failures.

### Project 1: `ors-system-3480b` — Main platform
- **Used by:** ORS-1/2/3/FC/R, Admin, Tracker (user data), SVR user access checks
- **Config:** `src/firebase.js`
- **Exports:** `db`, `auth`, `gProvider`
- **Collections:** `ors-users/{uid}`, `sites/{siteId}`, `ors-projects/ors_registry`

### Project 2: `ors-demand-platform` — Transaction Pipeline
- **Used by:** Deal Board · Tracker · Land Map
- **Config:** `src/modules/DealBoard/firebase.js`
- **Maps API key:** `AIzaSyCON67DgHi7fBa0C3TpbEPuAB4FwhsnIY8`

⚠️ **LandMap Maps API requires all four referrers in Google Cloud Console (Balaji action):**
```
haanest.app    haanest.app/*    www.haanest.app    www.haanest.app/*
```
Root cause: `www.haanest.app` was missing. `_mapReadyLM` callback never fires when referrer blocked. Code fix: 8-second timeout now rejects with a clear error message.

### Project 3: `evaluate-6f1bf` — Evaluate + Evaluate-SVR
- **Used by:** Evaluate module (both Evaluate and SVR tabs)
- **Config:** `src/modules/Evaluate/firebase.js`
- **Key:** `AIzaSyDChmWPMBUn_SCqfdfXFT_zinmspmZA0vs`
- **Exports:** `dbEval`, `authEval` (own auth instance — CRITICAL)

```js
// CORRECT — in src/modules/Evaluate/index.jsx:
import { dbEval, authEval, REPORTS_COLLECTION } from './firebase'
import { auth as authMain, db as dbMain } from '../../firebase'

// WRONG — was the root cause of "Missing or insufficient permissions":
import { auth as authEval } from '../../firebase'  // wrong project's auth
```

**Why it broke:** Firestore on `evaluate-6f1bf` checks `request.auth` — when `authEval` pointed to `ors-system-3480b`, the token was from the wrong project and was rejected. Fixed: `evaluate-6f1bf/firebase.js` now exports its own `getAuth(app)`.

**Evaluate Firestore rules (apply in Firebase Console → evaluate-6f1bf):**
```
match /land_reports/{reportId} { allow read, write: if request.auth != null; }
```

---

## 5. URL Routing — `src/shared/router.js`

No React Router. Manual state in `App.jsx`.

| URL Path | Screen ID | Module | Document Title |
|----------|-----------|--------|----------------|
| `/` | null | LandingPage / HomePage | haanest — Industrial Land Transactions |
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

**`src/shared/appUrls.js`** — derives `APP_URLS` map from router.js. Used by LandingPage tiles for new-window opens (`window.open(_blank)`). Never put APP_URLS inside `App.jsx` — causes circular import.

**vercel.json:** `{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }` — catch-all for SPA.

**Navigation behaviour:**
- LandingPage tile → `window.open(APP_URLS[id], '_blank')` — new tab
- Hamburger switch → `pushPath(path)` + `setScreen(id)` — in-window, Back/Forward work
- Refresh → Vercel → `index.html` → `pathToScreen(pathname)` → correct module
- Home → `replacePath('/')` + `setScreen(null)`

---

## 6. AppShell (`src/components/AppShell.jsx`)

Universal header — 52px, white `#fff`, `border-bottom: 1px solid #e8e8e4`.

**Left side:** Brand block only — `haanest-wordmark-trans.png` (22px) + BETA badge + tagline.
No breadcrumb (Home / module-id / module-name removed May 2026). Navigation via hamburger only.

**Tagline — single line:** `Know Ground Reality — A HONEST Report`
The second line "Verification Report: Site | Title | Seller" was removed from ALL headers May 2026. It appears only in content sections and card subtitles.

**Right side:** `actions` slot + UniversalSearch + HamburgerMenu

**DealBoard search in `actions`:** DealBoard passes a compact flat search input via the `actions` prop. This replaced the old floating `#floating-search` bar.

---

## 7. Homepage (`src/pages/HomePage.jsx`)

**Section order:**
1. Nav — white, `haanest-wordmark-trans.png`, Sign in, hamburger
2. **Hero S1 — Evaluate** — pine green `#01796F`
   - Title: "Institution or Individual: Protect Your Capital. Save Your Invaluable Time."
   - "Evaluate Your Land" highlighted: 20px, font-weight 900, white
   - CTA: **EVALUATE MINE** (flat, `borderRadius: 0`)
3. **Evaluate detail strip** — off-white, report card + copy
4. **Output** — "One honest report — before any commitment" + sample report card
5. **Trust Bar** — pine green, "Half Acre or Hundred Acres. Be Sure of Your Investment."
6. **Hero S2 — Title Analysis** — dark teal `#014a45`
   - Title: "The Blueprint for Transaction Readiness."
   - CTA: **Analyse My Land Title**
7. **Title Analysis description** — 3-col cards (Ownership / Reconciliation / Succession) + Report strip
8. **Hero S3 — Land Sourcing** — dark teal `#014a45`
   - "Green Field" in `#01796F`, "Brown Field" in `#8B7355`
   - Title: "Like it or Not, Real Estate is a Minefield. We Clear the Path from Finding to Buying."
   - CTA: **Source My Land**
9. **Stage table** — 5 rows: Stage | The Pain | The HAAnest Solution
10. Footer — pine green

**Rules:** All CTAs flat (`borderRadius: 0`). No black backgrounds. Mobile-responsive (all grids collapse ≤768px via injected CSS classes `hp-two-col`, `hp-three-col`, `hp-source-grid`, `hp-hero-sect`).

---

## 8. LandingPage (`src/pages/LandingPage.jsx`)

Post-login dashboard. White nav, Sign Out button (explicit), hamburger with dark bars (visible on white bg).
App tiles open in **new window** via `window.open(APP_URLS[id], '_blank')`.
Includes Evaluate-SVR card (URL: `/svr`).

---

## 9. Evaluate Module (`src/modules/Evaluate/`)

### Key files
- `index.jsx` (~2000 lines) — all views + components in one file
- `firebase.js` — own Firebase config for `evaluate-6f1bf`; exports `dbEval`, `authEval`
- `styles.js` — EVAL_CSS string (injected into DOM)
- `distanceEngine.js` — infrastructure distance calculations

### Components inside index.jsx

| Component | Purpose |
|-----------|---------|
| `EvalHeader` | Own white header (wordmark, tagline, hamburger, sign-out) |
| `UserDashboard` | Overlay — user's report stats |
| `CustomFieldEditor` | Admin adds per-site custom fields to S06/S07 |
| `CustomFieldDisplay` | Renders custom fields in InputView |
| `SVRCustomField` | Renders per-field response in SVRView (toggle + notes) |
| `NavTab` | Tab button with active underline |
| `SVRView` | Evaluate-SVR field app |
| `AdminView` | User management + site assignment |
| `ReportsView` | Reports dashboard |
| `InputView` | 8-section input form |
| `PreviewView` | Print preview + PDF download button |

### Views (role-based tabs)

| View | Who sees it |
|------|-------------|
| `reports` | Admin + Eval users (default) |
| `input` / `preview` | Admin + Eval users |
| `svr` | Admin + SVR users (SVR-only auto-routed here on login) |
| `admin` | Admin only (superadmin or platform_manager) |

**Permission flags:**
```js
const isAdmin   = orsProfile?.role === 'superadmin' || !!orsProfile?.platform_manager
const isSvrUser = !!orsProfile?.svr_access && !isAdmin
const isEvalUser= !!orsProfile?.eval_access || isAdmin
```

### Auth flow — TWO useEffects required

```js
// Effect 1: mount — watches evaluate-6f1bf auth
useEffect(() => {
  onAuthStateChanged(authEval, user => {
    if (user) {
      setAuthUser(user)
      if (isSuperAdmin) loadReports()
      else if (orsProfile) { validate → loadReports() }
      // else: wait for orsProfile useEffect below
    }
  })
}, [])

// Effect 2: fires when orsProfile arrives (async from ors-system-3480b)
useEffect(() => {
  if (authUser && orsProfile && allowed) loadReports()
}, [orsProfile])
```

Both are needed — orsProfile is async and arrives after mount.

### Form sections

| # | Name | Key fields |
|---|------|-----------|
| S01 | Site Identification | siteId, siteCoordinate, locationCircle, landSize |
| S02 | Location | district1/2, taluk1/2, village1/2 |
| S03 | Accessibility | highwayDistance, roadPattern, roadWidth |
| S04 | Infrastructure | auto-calculated distances via distanceEngine.js |
| S05 | Site details | additional info |
| S06 | **Site Verification Findings** | 14 Yes/No/Not Sure toggles + custom fields |
| S07 | **Compliance Findings** | 12 Required/Not Required/Maybe toggles + panChamiLand + Report By |
| S08 | Additional info | freeform |

**Report By fields (S07 bottom):** `siteVerificationBy`, `siteVerificationContact`, `complianceVerificationBy`, `complianceVerificationContact`

### Custom fields (per-site, admin-defined)

- Admin creates fields before assigning SVR user
- Types: `yesno` (Yes/No/Not Sure), `reqd` (Required/Not Required/Maybe), `text`
- Stored: `customFieldsS06: [{id, label, type, options}]` on `land_reports` doc
- Answers: `form[fieldId]` + `form[fieldId+'_note']`
- SVR answers: `svr_cf_{fieldId}` + `svr_cf_{fieldId}_note`
- Appear in: InputView, Preview, PDF, SVRView (auto-synced live)

### PDF download

Built from `form` state (not DOM). Old bug: checked `document.getElementById('pdf-preview-content')` which didn't exist → "preview not ready". Fixed: DOM check removed, always builds from form state.

### Prefill from Tracker

Evaluate reads URL params on mount:
```js
// /evaluate?siteId=LB-260503-XXXXX&coord=12.8,79.9&village=Paranur&acres=8.4
→ prefills S01 fields, opens Input view, cleans URL via history.replaceState
```

### `land_reports` Firestore schema

```js
{
  siteId, siteCoordinate, locationCircle, landSize,
  district1, taluk1, village1, district2, taluk2, village2,
  // S06
  officialCategory, siteFrontage, siteLevel,
  encroachment, htTowerLine, htTowerBase, waterChannelRevenue,
  waterBodyRevenue, waterBodyOnsite, govtLand, landAcquisition,
  landCeiling, landReforms, quarry, burial, asi, forest,
  // S07
  landUseConversion, panChamiLand, conversionExtent, pwdPermission,
  courtOrder, swappingAnandheenam, swappingTemple, sipcotNoc,
  highwaysNoc, asiNoc, ongcNoc, acquisitionNoc, otherConditions,
  // Custom fields
  customFieldsS06: [{id, label, type, options}],
  customFieldsS07: [{id, label, type, options}],
  // Report By
  siteVerificationBy, siteVerificationContact,
  complianceVerificationBy, complianceVerificationContact,
  additionalInfo,
  // SVR
  assignedSvrEmail, assignedSvrUid, assignedAt,
  svrGeoCapture, svrGeoTime,
  svrEncroachment, svrHtTowerLine, svrHtTowerBase, svrWaterChannel,
  svrWaterBody, svrWaterBodyOnsite, svrGovtLand, svrLandAcquisition,
  svrLandCeiling, svrLandReforms, svrQuarry, svrBurial, svrAsi,
  svrForest, svrPanChamiLand, svrOfficialCategory, svrSiteFrontage,
  svrSiteLevel, svrNotes, svrCompletedBy, svrCompletedAt, svrUpdatedAt,
  // svr_cf_{fieldId}, svr_cf_{fieldId}_note for custom field SVR responses
  createdAt, updatedAt,
}
```

---

## 10. Evaluate-SVR

**URL:** `haanest.app/svr` → Evaluate module with `initialView='svr'`
**Role:** `svr_access: true` on `ors-users`

**Field flow:**
1. Site list (filtered to assigned sites; admin sees all)
2. Auto-populated S01 info read-only
3. 📍 Capture Current Location → `svrGeoCapture` + `svrGeoTime`
4. S06 fields (Yes/No/Not Sure) + custom fields (toggle + notes)
5. S07 custom fields (if admin defined)
6. Field Notes textarea
7. Save → `svr_*` fields written to `land_reports` → Evaluate sees via `onSnapshot`

---

## 11. Evaluate Admin View

**Access:** `isAdmin` only (superadmin or platform_manager).

**Tab 1 — User Management:**
- Table: User/Email · Status · Role · Evaluate · SVR Field · Platform Mgr · Actions
- Status: ACTIVE / ON HOLD / FROZEN — ACTIVATE / HOLD / FREEZE buttons
- One-click privilege toggle chips (`eval_access`, `svr_access`, `platform_manager`)
- Add User form → creates `ors-users` doc on `ors-system-3480b`
- Search by name/email
- Uses `dbMain` (static import) — NOT dynamic import (dynamic was the permissions bug)

**Tab 2 — Site Assignment:**
- Assign active SVR users to reports
- Dropdown filters to `status === 'active'` SVR users only
- Writes `assignedSvrEmail`, `assignedSvrUid`, `assignedAt` to `land_reports`

---

## 12. Deal Board (`src/modules/DealBoard/`)

**Firebase:** `ors-demand-platform`

**Search moved to AppShell header (May 2026):**
- `dbSearch` state in `DealBoard/index.jsx`
- Rendered via `actions` prop: compact flat input, magnifier icon, × clear
- Passed as `externalSearch` to `DemandsListView`
- Old floating `#floating-search` div removed from `DemandsListView.jsx`

**Layout offsets (no floating search):**
```css
.demands-layout { padding-top: 116px; }  /* AppShell 52 + DB header 60 + 4 breathing */
.sidebar        { top: 112px; height: calc(100vh - 112px); }
.sidebar-right  { top: 112px; height: calc(100vh - 112px); }
```

---

## 13. Field Connect (`src/modules/ORSFC/`)

**Served as full-screen iframe** from `/public/fieldconnect.html`.

Original app: 1,644-line vanilla JS, Firebase compat SDK, connects to `ors-system-3480b`, full CRUD (families, members, land records, meetings), email/password auth.

haanest theme applied to HTML: `#1a73e8` → `#01796F`, login wordmark, flat buttons, title updated.

**React wrapper:** `AppHeader` (52px, hamburger) + `<iframe src="/fieldconnect.html" allow="geolocation">` filling remaining viewport.

---

## 14. Tracker (`src/modules/Tracker/`)

**Firebase:** `ors-demand-platform`

**Evaluate button on every site card (`SiteCard.jsx`):**
```js
function openEvaluate(s) {
  const params = new URLSearchParams()
  if (s.siteId)       params.set('siteId',   s.siteId)
  if (s.lat && s.lng) params.set('coord',    s.lat+','+s.lng)
  if (s.village)      params.set('village',  s.village)
  if (s.landmark)     params.set('landmark', s.landmark)
  if (s.acres)        params.set('acres',    s.acres)
  if (s.district)     params.set('district', s.district)
  if (s.taluk)        params.set('taluk',    s.taluk)
  window.open(`${window.location.origin}/evaluate?${params.toString()}`, '_blank')
}
```

---

## 15. LandMap (`src/modules/LandMap/`)

**Maps not loading — root cause:** `RefererNotAllowedMapError`. Script loads (HTTP 200) but `_mapReadyLM` callback never fires. Old code hung silently.

**Code fix:** 8-second timeout — rejects with actionable error message if callback doesn't fire.

**Balaji console action required:**
Google Cloud Console → key `AIzaSyCON67DgHi7fBa0C3TpbEPuAB4FwhsnIY8` → HTTP referrers → add:
```
haanest.app    haanest.app/*    www.haanest.app    www.haanest.app/*
```

---

## 16. User Permissions (`ors-users/{uid}` on `ors-system-3480b`)

**Key fields (May 2026):**
```js
status:           'active' | 'hold' | 'frozen',
eval_access:      boolean,
svr_access:       boolean,
platform_manager: boolean,
db_role:          '' | 'admin' | 'agent' | 'client' | 'viewer',
ors_apps:         string[],   // ['ORS-1','ORS-2','ORS-3','ORS-FC','ORS-R']
ors_projects:     string[],   // empty = all sites
validUntil:       string | null,
role:             'superadmin' | 'admin' | 'user',
```

**Access rules:**

| Module | Check |
|--------|-------|
| `EVAL` | `eval_access === true` OR superadmin |
| `SVR` | `profile?.svr_access OR eval_access OR platform_manager` — checked DIRECTLY, NOT via `canAccess('SVR')` which has no SVR rule and always returns false |
| `ADMIN (Evaluate)` | `platform_manager === true` OR superadmin |
| `DB` / `TR` | `db_role` non-empty |
| `MAP` | `db_role === 'admin'` OR `'agent'` |
| `ORS-1/2/3/FC/R` | `ors_apps` includes that ID |
| `ADMIN` | `role === 'admin'` OR superadmin |

**Real accounts:**

| Email | Role |
|-------|------|
| `balaji@lakshmibalajio2o.com` | superadmin (hardcoded — never revoked) |
| `ejaz_nathani@welspun.com` | Welspun developer |
| `raj.attri@ccigroup.co.in` | CCI developer |

---

## 17. Stage 2 — Title Analysis Modules

All served via `src/modules/SiteWorkspace/index.jsx`. Data lives in `sites/{siteId}` on `ors-system-3480b`.

| Module | Screen ID | Path | Purpose |
|--------|-----------|------|---------|
| ORS-1 Ownership | `ORS-1` | `/ownership` | Families, members, land records, transfers, meetings |
| ORS-2 Reconciliation | `ORS-2` | `/reconciliation` | 12+ automated gap checks, survey tree, area accounting |
| ORS-3 Succession | `ORS-3` | `/succession` | Legal heirs, Class I/II hierarchy, dormant claimants |
| ORS-FC Field Collect | `ORS-FC` | `/fieldconnect` | On-site document collection (iframe) |
| ORS-R Title Report | `ORS-R` | `/report` | Transaction readiness + reconciliation + succession scores |

`PERSONA_META` must stay in `src/modules/Admin/personas.js` — never move into `Admin/index.jsx` (causes TDZ crash).

---

## 18. Technical Patterns

### Python3 for multi-line JSX edits
```bash
python3 << 'PYEOF'
with open('/home/claude/ors-platform/src/path/file.jsx', 'r', encoding='utf-8') as f:
    c = f.read()
old = """exact multi-line string"""
new = """replacement"""
if old in c:
    c = c.replace(old, new); print("OK")
else:
    print("MISS — check exact whitespace")
with open('/home/claude/ors-platform/src/path/file.jsx', 'w', encoding='utf-8') as f:
    f.write(c)
PYEOF
```

### Byte-level for styles.js (CSS-in-JS with escaped newlines)
```python
with open('styles.js', 'rb') as f: raw = f.read()
raw = raw.replace(b'old css\\n    rule', b'new css\\n    rule')
with open('styles.js', 'wb') as f: f.write(raw)
```

### Always verify before editing
```bash
grep -n "exact string" src/path/file.jsx
# 0 results → stop. 1 → safe. 2+ → identify which.
```

### Always build before push
```bash
cd /home/claude/ors-platform && npm run build
# Must show: ✓ built in X.XXs
# Pre-existing warnings (chunk size, ui.jsx duplicate key) are safe to ignore
```

### Standard workflow
```bash
# Clone fresh container
git clone https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git /home/claude/ors-platform
cd /home/claude/ors-platform && npm install

# Edit → build → commit → push
git config user.email "balaji@lakshmibalajio2o.com"
git config user.name "Balaji"
git add -A && git commit -m "fix/feat: description" && git push origin main
# Vercel deploys in ~60 seconds
```

---

## 19. Known Issues & Pending

| Item | Status |
|------|--------|
| LandMap Maps API | `www.haanest.app/*` not yet added to referrers — Balaji must do in Google Cloud Console |
| `canAccess('SVR')` | Always returns false by design — check `profile?.svr_access` directly |
| Admin personas.js | `PERSONA_META` must never move to `Admin/index.jsx` — TDZ crash |
| Evaluate-SVR photo upload | Not built — toggle + text notes only |
| Evaluate-SVR offline/PWA | Not built |
| Predictive Analytics | Marked "Coming Soon" — Gemini 1.5-flash deprecated |

### Pre-existing build warnings (safe to ignore)
- Chunks > 500kB
- Duplicate key in `src/shared/ui.jsx`
- Dynamic import of `Evaluate/firebase.js`

### Colour audit command
```bash
grep -rn "#1a3a5c\|#1d9bf0\|#6141ac" src/ --include="*.jsx" --include="*.js" |
  grep -v "DealBoard/styles.js\|LandMap\|Tracker\|global.css"
# DealBoard/Tracker/LandMap styles.js intentionally retain old colours
```

---

*Update this file at the end of every session.*

---

## 20. 13 May 2026 — Full Platform Upgrade Session

### User Management (Evaluate AdminView)

**8 new role profiles** in Add User form:
`admin` · `agent` · `client` · `demand-assistant` · `demand-manager` · `supply-manager` · `transaction-partner` · `custom`

**New fields:** Company · Phone · Access Valid Until (time-bound, enforced by `canAccess`) · Internal Notes

**Demand assignment per user:** Dropdown in each user row → saves `assignedDemandId` to `ors-users`. × REVOKE button removes assignment.

**New `ors-users` fields written on create:**
```js
company, phone, db_role, validUntil, notes,
firstLogin: true, termsAccepted: false
```

### Terms of Use (first login gate)

After first login, before any screen, `TermsModal` blocks with LBR terms text. User must checkbox + "I Agree". On accept: `termsAccepted: true, firstLogin: false` written to `ors-users`. Immediately followed by `ChangePasswordModal`.

### Password Change (first login)

After terms accepted, `ChangePasswordModal` prompts user to set a password (min 8 chars). Uses `firebase/auth updatePassword()`. Skip option available — can use "Forgot Password" on next login.

### LoginModal — haanest brand

Colours updated from `#1a3a5c`/`#1d9bf0` → `#01796F`/`#8B7355`. Wordmark PNG replaces text. Wording: "Sign in with your official email".

### Client role — demand-specific access

`db_role === 'client'` users in Tracker:
- Only see **Demand Specific Sites** tab (all other tabs hidden)
- Filtered to `orsProfile.assignedDemandId` — their demand only
- `isClient` flag detected in Tracker from `orsProfile.db_role`

### LandMap access

`canAccess('MAP')` now returns true for ANY non-empty `db_role` (was admin/agent only). All 8 new roles can access LandMap.

### Subdomain routing

```js
siteoptions.haanest.app → Tracker (initialTab: 'TR')
sitesmap.haanest.app    → LandMap (initialTab: 'MAP')
```
Detected via `window.location.hostname.startsWith('siteoptions.')` in App.jsx.
**Vercel action needed:** Add `siteoptions.haanest.app` and `sitesmap.haanest.app` as custom domains on the same Vercel project (same build, no separate deployment needed).

### Site Validation Report

New component: `src/modules/Tracker/views/SiteValidationReport.jsx`
Rendered under each site card in Demand Specific Sites tab.
Stored in: `ors-demand-platform / site-validations / {siteId}`

Fields:
- **Feasibility** — Feasible / Not Feasible / Maybe (header bar changes colour)
- **Sale By** — Owner / PoA Agent / Agreement Holder / Aggregator / Other
- **4 Yes/No/Maybe toggles** — Extent Willing · Portion Willing · Truck Access · Operational Feasibility
- **Sale Terms / Challenges / Risks** — freeform text
- **Comments** — any user can post; stored as arrayUnion

Permissions: Admin/Manager = full edit. Client = read-only + comment.

### SiteCard — map pin hidden

"📍 View on Map" link removed from Tracker site cards. Coordinates still stored and available in Edit modal and LandMap.

### Tracker role detection

```js
const userRole = orsProfile?.db_role || ''
const isManager = ['admin','demand-manager','supply-manager'].includes(userRole) || isAdmin
const isClient  = userRole === 'client'
const canViewAllSites = isAdmin || ['agent','demand-assistant','demand-manager','supply-manager','transaction-partner'].includes(userRole)
const assignedDemandId = orsProfile?.assignedDemandId || null
```

### Firestore rules — ors-demand-platform

Added `site-validations` collection: `allow read, write: if true`
**Apply in Firebase Console → ors-demand-platform → Firestore → Rules.**

### Pending from requirements

- `siteoptions.haanest.app` and `sitesmap.haanest.app` — **Balaji must add as custom domains in Vercel Dashboard** (Project Settings → Domains → Add)
- Site Feasibility detailed report (separate full-page view)
- Customer portal (separate auth tier for clients)
- Demand-based assignment UI (full relational view)
