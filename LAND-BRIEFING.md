# land.orsone.app — Claude Briefing Document
**Repo:** `LBRBalaji/ors-trace-origin` (private)  
**Live:** [land.orsone.app](https://land.orsone.app)  
**Last updated:** April 2026  
**PAT:** `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

> Read this before touching any file. Verify with `grep -n` before every edit.
> Never assume — always confirm the exact string exists first.

---
> **Note:** The GitHub PAT is redacted in this public document. Balaji has the actual token.
> When starting a session, provide the PAT separately or retrieve it from the private repo.



## 1. What is land.orsone.app?

An integrated React/Vite web platform for industrial land real estate transactions in India. Built and operated by **Lakshmi Balaji ORS Private Limited**, Chennai. Owner: **Balaji Aram Valartha Sundaram** (non-developer — directs all development).

**Mission:** Take a land transaction from raw market sourcing through site evaluation, title analysis and seller verification — in one integrated platform.

**The four-step land transaction journey:**
1. **Source Land** — post demand, receive structured site submissions from market
2. **Site Verified** — threats, risks and challenges assessed (Evaluate module)
3. **Title Verified** — ownership records, reconciliation and succession analysis
4. **Sellers Verified** — seller identity and authority confirmed

**Two stages, nine modules:**

| Stage | Modules |
|-------|---------|
| Stage 1 — Sourcing & Evaluation | Deal Board · Tracker · Land Map · Evaluate |
| Stage 2 — Title Analysis | Ownership (ORS-1) · Reconciliation (ORS-2) · Succession (ORS-3) · Field Collect (ORS-FC) · Title Report (ORS-R) |

---

## 2. Stack

| Layer | Detail |
|-------|--------|
| Framework | React 18 + Vite 5 |
| Language | JavaScript (JSX) — no TypeScript anywhere |
| Styling | Inline styles only — CSS-in-JS via `T` tokens object. One `global.css` for resets only |
| Auth | Firebase Auth (email/password). Super admin only: `balaji@lakshmibalajio2o.com` |
| Package manager | npm |
| Hosting | Vercel — auto-deploys on `git push origin main` (~60 seconds) |
| Repo | `LBRBalaji/ors-trace-origin` (private) |
| Remote URL | `https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git` |

**Dependencies** (package.json):
```json
"dependencies": {
  "firebase": "^10.12.2",
  "react": "^18.3.1",
  "react-dom": "^18.3.1"
}
```
No other libraries. No Redux, no router, no CSS framework.

---

## 3. Three Firebase Projects

This is the most critical architectural fact. **Three separate Firestore databases.** Getting this wrong causes silent data load failures.

### Project 1: `ors-system-3480b` — Main platform
**Used by:** ORS-1, ORS-2, ORS-3, ORS-FC, ORS-R, Admin, Universal Search  
**Config:** `src/firebase.js` (env vars via Vite `VITE_FIREBASE_*`)  
**Region:** asia-south1

**Collections:**
| Collection | Contents |
|------------|----------|
| `ors-users/{uid}` | All user profiles, permissions, validity |
| `sites/{siteId}` | All Title Analysis site data (families, members, land records, transfers, siteAssets, digitalAssets, meetings, clubbedMeasurements) |

**Site document schema** (`sites/{siteId}`):
```js
{
  name:                string,          // site display name
  createdAt:           ISO string,
  families:            Family[],
  members:             Member[],
  landRecords:         LandRecord[],
  transfers:           Transfer[],
  meetings:            Meeting[],
  siteAssets:          { sitesketch:[], villagemap:[], locationmap:[], other:[] },
  digitalAssets:       { /* legal documents by type */ },
  clubbedMeasurements: [],
}
```

### Project 2: `ors-demand-platform` — Transaction Pipeline
**Used by:** Deal Board · Tracker · Land Map  
**Config:** `src/modules/DealBoard/firebase.js`  
**Key:** `AIzaSyAssF4cjQCOuY-Ww6mEBIUMd78rJH8g_gY`

**Collections:**
| Collection | Contents |
|------------|----------|
| `demands` | All land acquisition demands posted on Deal Board |
| `propertySubmissions` | Properties submitted against demands |
| `clients` | Client/tenant records |
| `activity` | Activity log entries |
| `sites` | Tracker pipeline sites (different from ors-system-3480b sites) |
| `trackerUsers` | Tracker user list |
| `publicStats` | Aggregated demand counts for Deal Board sidebar |

### Project 3: `evaluate-6f1bf` — Evaluate module
**Used by:** Evaluate module only  
**Config:** `src/modules/Evaluate/firebase.js`  
**Key:** `AIzaSyDChmWPMBUn_SCqfdfXFT_zinmspmZA0vs`  
**Export:** `dbEval` (NOT `db`)

**Collections:**
| Collection | Contents |
|------------|----------|
| `land_reports` | All Evaluate preliminary assessment reports |

**CRITICAL:** Always import `dbEval` from `./firebase` inside the Evaluate module.  
**NEVER** import `db` from `../../firebase` for Evaluate data — that reads from ors-system-3480b (empty).

---

## 4. Design System

### Single source of truth: `src/shared/tokens.js`
Import as `import { T } from '../shared/tokens'` (adjust path depth).  
**Never hardcode hex values in components.** Always use `T.purple`, `T.navy`, etc.

### Three strict theme groups — NEVER cross-apply

**Group 1 — Deal Board · Tracker · Land Map · Evaluate**
```
Fonts:      DM Sans (body), Space Mono (labels/IDs)
            Injected via Google Fonts in each module's style injection
Navy:       #1a3a5c   (header bg, primary buttons)
Blue:       #1d9bf0   (accent, links, chip borders, active states)
Page bg:    #fafaf8   (warm off-white)
Muted text: #536471
Border:     #e8e8e4
```

**Group 2 — ORS-1/2/3/FC/R · Admin (Title Analysis)**
```
Font:       Helvetica Neue / Arial (no monospace, no DM Sans)
Primary:    T.purple = #1d9bf0  (electric blue — was #6141ac purple before April 2026)
Navy:       T.navy = #1a3a5c    (header background)
Page bg:    T.page = #fafaf8    (was lavender hsl(259,30%,96%) before April 2026)
Border:     T.border = #e8e8e4
NavyText:   T.navyText = #0f1419 (body text)
PurpleLight: T.purpleLight = rgba(29,155,240,0.10) (chip backgrounds)
```

**Group 3 — Evaluate**
- Uses DM Sans + Space Mono (same as Group 1)
- But colour palette is same as Group 2 (#1a3a5c, #1d9bf0)
- Has its own Firebase project

### AppShell constants (`src/components/AppShell.jsx`)
```js
const NAVY    = '#1a3a5c'   // universal header background
const PURPLE  = '#1d9bf0'   // active item highlights (named PURPLE, value is blue)
const PURPLEM = 'rgba(29,155,240,0.65)'
```

### Design rules (enforced always)
- No emojis in UI — ever
- No icons with "O in a square box" logo
- No hardcoded hex values — always `T.purple`, `T.navy` etc.
- Header: single line — "ORS-ONE" + breadcrumb + module badge
- Footer: "Lakshmi Balaji ORS Private Limited" left, "Building Transaction Ready Assets" centre
- Zoho-inspired UX pattern: filter tabs → ZohoTable → row-click → 3-col record view
- `global.css` has old purple CSS variables — they are **not used** by React components (which all use inline styles via `T`). Do not update global.css — it is legacy only.

---

## 5. Routing & Navigation

All routing is **manual state management** in `App.jsx` — no React Router.

```
App.jsx
│
├─ isLoading → Spinner
│
├─ !isLoggedIn
│   ├─ accessDenied = 'pending_approval' → PendingScreen
│   ├─ accessDenied = 'revoked'          → RevokedScreen
│   ├─ accessDenied = 'error'            → ErrorScreen
│   ├─ screen = 'contact'                → ContactPage
│   ├─ productId set                     → ProductPage (+ optional LoginModal)
│   └─ default                           → HomePage (+ optional LoginModal)
│
└─ isLoggedIn
    ├─ screen = null        → LandingPage (personalised dashboard)
    ├─ screen = 'contact'   → ContactPage
    ├─ screen = 'ORS-1'     → SiteWorkspace (initialModule='ORS-1')
    ├─ screen = 'ORS-2'     → SiteWorkspace (initialModule='ORS-2')
    ├─ screen = 'ORS-3'     → SiteWorkspace (initialModule='ORS-3')
    ├─ screen = 'ORS-FC'    → ORSFC
    ├─ screen = 'ORS-R'     → SiteWorkspace (initialModule='ORS-R')
    ├─ screen = 'DB'        → DealBoard
    ├─ screen = 'TR'        → Tracker
    ├─ screen = 'MAP'       → LandMap
    ├─ screen = 'EVAL'      → Evaluate
    ├─ screen = 'ADMIN'     → Admin (superadmin / admin role only)
    └─ productId set        → ProductPage (logged-in context, shows Open App button)
```

**`navigate(id)` function** (passed as `onNavigate` through all modules):
- `null` or `'CONTACT'` → ContactPage / Home
- IDs starting with `'TA-'` → ProductPage (product info) first
- `'ORS-R'` → SiteWorkspace with initialModule='ORS-R'
- All other IDs → direct module screen

**Module access guard:** If `canAccess(screen)` returns false → `AccessDeniedScreen`

---

## 6. Module Inventory

### `src/pages/HomePage.jsx` (728 lines)
Pre-login public marketing page. Zoho-inspired scroll reveal. Sections: hero → how it works (Stage 1 + Stage 2) → app suite → who it's for → two paths CTA (DIY vs Managed Service). Registration form sends via WhatsApp.

### `src/pages/LandingPage.jsx` (434 lines)
Post-login personalised dashboard. Shows: time-aware greeting ("Good morning, [name]") → accessible app tiles grouped by stage → locked apps shown dimmed with "No access" label → hamburger with quick links to accessible apps only.

### `src/components/LoginModal.jsx` (267 lines)
Modal overlay (not a page). Navy/blue theme. Email + password. Show/hide password toggle. Forgot password flow (sends Firebase reset email). Backdrop + ESC to close.

### `src/components/AppShell.jsx` (300 lines)
Universal header used by all modules. Contains: ORS-ONE logo, breadcrumb (module › section), hamburger menu (permission-filtered, shows only accessible apps), Search (Ctrl+K). Constants: NAVY, PURPLE, PURPLEM. `AppHeader` and `AppFooter` exported.

### `src/components/UniversalSearch.jsx` (380 lines)
Ctrl+K search across all 8 apps. Searches: ORS sites/families/members/surveys, Deal Board demands/submissions, Tracker sites, Evaluate reports, Users. Grouped results with match highlighting.

### `src/hooks/useAuth.js` (141 lines)
Firebase Auth listener. Key exports:
- `SUPER_ADMIN = 'balaji@lakshmibalajio2o.com'` — hardcoded
- `useAuth()` → `{ user, profile, accessDenied, canAccess, canAccessProject, isSuperAdminUser }`
- `canAccess(mod)` — checks `profile.ors_apps`, `db_role`, `eval_access`, `validUntil`
- `canAccessProject(siteId)` — checks `profile.ors_projects` (empty = all sites)
- Email-based profile lookup fallback + uid migration on first login
- `validUntil` auto-lock: if date is past, `canAccess` returns false (super admin exempt)

### `src/shared/tokens.js` (127 lines)
**Single source of truth for all design decisions.** All colours, spacing, type scale, table styles, KPI card styles. Must be imported as `T` and used in all components.

### `src/shared/ZohoTable.jsx` (343 lines)
Reusable Zoho-style data table. Props: `columns`, `rows`, `rowActions`, `onRowClick`, `emptyMessage`. Handles sorting, row hover, overflow action menu.

### `src/shared/ui.jsx` (279 lines)
Shared UI primitives: `Btn`, `Toast`, `ConfirmDialog`, `SearchInput`, `PageHeader`, `EmptyState`.

### `src/modules/SiteWorkspace/index.jsx` (567 lines)
Router for all Title Analysis modules (ORS-1/2/3/FC/R). Renders:
1. `ProjectSelector` (site card grid) when no site selected
2. The appropriate ORS module view once a site is selected
All Title Analysis data lives in Firestore `sites/{siteId}` on `ors-system-3480b`.

### `src/modules/ORS1/` — Ownership module
Entry: `ORS1/index.jsx` (161 lines) → passes site data to views via `useProjectData.js`

**Sidebar views:**
| ID | Label | File |
|----|-------|------|
| `dashboard` | Dashboard | `views/Dashboard.jsx` |
| `families` | Families | `views/Families.jsx` |
| `members` | Family Tree | `views/Members.jsx` |
| `landPedigree` | Land Pedigree | `views/LandPedigree.jsx` |
| `transfers` | Transfers | `views/Transfers.jsx` |
| `meetings` | Meetings | `views/Meetings.jsx` |
| `siteAssets` | Site Assets | `views/SiteAssets.jsx` |
| `digitalAssets` | Digital Assets | `views/DigitalAssets.jsx` |
| `admin` | Admin | `views/Admin.jsx` |

**Record views (Zoho 3-col pattern):**
- `FamilyRecord.jsx` (699 lines) — family detail + member list + land records
- `MemberRecord.jsx` (560 lines) — member detail + flags + heir class
- `LandRecord.jsx` (511 lines) — land record detail + EC/Patta
- `TransferRecord.jsx` (503 lines) — transfer detail + document chain

**Data hook:** `useProjectData.js` — reads from `sites/{siteId}` on main Firestore

### `src/modules/ORS2/` — Reconciliation module
Entry: `ORS2/index.jsx` (194 lines). Engine: `engine.js` (298 lines).

**Sidebar views:**
| ID | Label | File |
|----|-------|------|
| `dashboard` | Dashboard | `views/Dashboard.jsx` |
| `ledger` | Survey Ledger | `views/SurveyLedger.jsx` |
| `timeline` | Timeline | `views/Timeline.jsx` |
| `gaps` | Gap Analysis | `views/GapAnalysis.jsx` |
| `table` | All Records | `views/RecordsTable.jsx` |

**Record views:**
- `IssueRecord.jsx` (500 lines) — gap/issue detail with CHK-N code, Fix-in-Ownership links
- `SurveyChainRecord.jsx` (453 lines) — survey chain with area accounting box

**Engine (`engine.js`):** Builds survey tree from land records. Detects 12 gap types. Calculates reconciliation score (0–100). Resolves parent/child survey chains. Area accounting.

### `src/modules/ORS3/` — Succession module
Entry: `ORS3/index.jsx` (187 lines). Engine: `engine.js` (186 lines).

**Sidebar views:**
| ID | Label | File |
|----|-------|------|
| `dashboard` | Dashboard | `views/Dashboard.jsx` |
| `trees` | Family Trees | `views/FamilyTrees.jsx` |
| `heirs` | Heir Analysis | `views/HeirAnalysis.jsx` |
| `settlement` | Settlement Audit | `views/SettlementAudit.jsx` |
| `mitigation` | Mitigation Roadmap | `views/Mitigation.jsx` |

**Record views:**
- `HeirRecord.jsx` (573 lines) — heir detail with entitlement, flags, dormant status
- `PrintReport.jsx` (289 lines) — combined title analysis print report (opens new window)

**Engine (`engine.js`):** Applies Indian succession law. Identifies entitled heirs by Class I/II hierarchy. Handles Self-Acquired and Ancestral property types. Dormant claimant detection. Succession score calculation.

### `src/modules/ORSR/` — Title Report
`ORSR/index.jsx` (396 lines). Combined readiness report showing transaction readiness score, reconciliation score, succession score, risk flags, legal mitigation roadmap, and delivery model (DIY vs Managed Service).

### `src/modules/ORSFC/` — Field Collect
`ORSFC/index.jsx` (32 lines). Mobile-first data capture tool for on-site document and record collection.

### `src/modules/DealBoard/` — Deal Board
Entry: `DealBoard/index.jsx` (205 lines).  
Firebase: **`ors-demand-platform`** (separate project).  
Auth: uses main Firebase auth but reads/writes to demand-platform Firestore.

**Views:**
| File | Purpose |
|------|---------|
| `DemandsListView.jsx` | Main list — natural page scroll, sticky sidebars at top:156px |
| `AdminDemandForm.jsx` | Admin form — Google Maps Drawing Manager, Circle+Polygon tools |
| `DashboardView.jsx` | Stats dashboard |
| `SubmissionsView.jsx` | Property submissions received |
| `AgentPortalView.jsx` | Agent submission interface |
| `ClientInboxView.jsx` | Client view of matched properties |
| `ClientsView.jsx` | Client list management |
| `DemandReviewView.jsx` | Demand review and approval |
| `PropertySubmitModal.jsx` | Submit matching property modal |
| `UsersView.jsx` | User management within DealBoard |

**Auth gate logic:**
- Not signed in → toast + sign-in prompt
- Admin → AdminDemandForm
- Approved non-admin → AgentPortalView
- Pending → toast

**Scroll fix (critical):**  
Fixed layers: AppShell 52px + DealBoard header 60px + floating search 46px = **158px total**.  
`.demands-layout { padding-top: 160px }` — content offset below all fixed layers.  
Sidebars: `position: sticky; top: 156px; height: calc(100vh - 156px)`.  
Do not use `height: calc(100vh - Npx); overflow: hidden` on the outer wrapper — this breaks scroll.

### `src/modules/LandMap/` — Land Map
`LandMap/index.jsx` (674 lines).  
Firebase: **`ors-demand-platform`** for sites. Main Firebase for auth.  
Google Maps API key: `AIzaSyCON67DgHi7fBa0C3TpbEPuAB4FwhsnIY8`

**Features:**
- 5 road corridors: CPRR, CTE, NE-7 CBE Expressway, ORR, STRR — exact coordinates
- Infrastructure markers: ports (navy anchor), airports (teal plane), dry ports (indigo warehouse)
- Distance Calculator (Google Distance Matrix API)
- Map/Satellite toggle
- Toolbar: single row — search + type filter + category filter + TOTAL/SHOWN count
- Site markers with click-to-details

**Toolbar CSS fix (committed):**  
`.map-toolbar { height: 44px; overflow: visible; flex-shrink: 0; flex-wrap: nowrap }`  
`.map-search-wrap { flex: 1 1 200px; min-width: 160px }`  
`.filter-select { height: 32px; flex-shrink: 0 }`

### `src/modules/Tracker/` — Tracker
`Tracker/index.jsx` (416 lines).  
Firebase: **`ors-demand-platform`** for sites/trackerUsers.

Tracks land sites through acquisition pipeline. Site cards, status tracking, activity log, WhatsApp share.

### `src/modules/Evaluate/` — Evaluate
`Evaluate/index.jsx` (784 lines).  
Firebase: **`evaluate-6f1bf`** (`dbEval` from `./firebase`).  
Auth: main Firebase auth (unified).

**UI pattern:** Zoho-style landing (stats bar + sidebar filters + card feed).  
**Card feed:** Space Mono IDs, Complete/In-Progress badges, 4-col chip grid, Open/Preview/Delete actions.  
**Views:** Saved Reports → Input Form → Preview  
**Distance engine:** `distanceEngine.js` — calculates distances to INFRA_COORDS (ports, airports, dry ports).

**Data loading fix:** Two `useEffect` hooks:
1. `[]` — fires on mount with auth state
2. `[orsProfile]` — re-fires when orsProfile loads (async, arrives after mount)
Both are needed because the auth state resolves before the orsProfile is fetched.

### `src/modules/Admin/` — User Management
Entry: `Admin/index.jsx` (432 lines).

**List view:** Stat header (All/Active/Pending/Expiring/Revoked as numbered cards) → search bar → rich table. Hover reveals contextual actions: ✓ Approve / ↩ Restore / ✕ Revoke.

**UserRecord:** `views/UserRecord.jsx` (619 lines). 3-col layout:
- Left: identity (avatar, editable name, user type, company/phone, timestamps)
- Centre: tabs — App Access (per-app cards with toggles and role selectors) · Site Access (chip input) · Details (validity, notes)
- Right: context-aware primary action + access summary

**UserForm:** `views/UserForm.jsx` (367 lines). Stepped layout:
- Left: 5 numbered sections (Identity → Transaction Pipeline → Title Analysis → Site Access → Validity)
- Right: sticky live Access Preview panel

**personas.js:** `Admin/personas.js` (17 lines). Extracted to break circular import TDZ crash.  
⚠️ **Never move PERSONA_META back into Admin/index.jsx** — it will cause runtime TDZ crash.

---

## 7. User System

### Firestore: `ors-users/{uid}` (on `ors-system-3480b`)

```js
{
  // Identity
  email:               string,           // lowercase always
  displayName:         string,
  company:             string,
  phone:               string,
  role:                'superadmin' | 'admin' | 'user',

  // Persona
  persona:             'internal' | 'partner' | 'aggregator' | 'service' | 'client' | 'other',

  // Access flags
  active:              boolean,          // false = revoked
  approvedBy:          string,           // email of approver
  approvedAt:          string,           // ISO date

  // Module permissions
  db_role:             '' | 'admin' | 'agent' | 'client' | 'viewer',
  eval_access:         boolean,
  ors_apps:            string[],         // ['ORS-1','ORS-2','ORS-3','ORS-FC','ORS-R']
  ors_projects:        string[],         // site IDs — empty array = all sites

  // Validity
  validUntil:          string | null,    // ISO date — access auto-cut on expiry
  validityReminderDays: number,          // days before expiry for warning (default 14)

  // Admin
  notes:               string,
  uid:                 string,           // Firebase Auth UID
  createdAt:           string,
  updatedAt:           string,
}
```

### Access rules (`canAccess` in useAuth.js)

| Module ID | Requires |
|-----------|---------|
| `ORS-1` | `ors_apps` includes `'ORS-1'` |
| `ORS-2` | `ors_apps` includes `'ORS-2'` |
| `ORS-3` | `ors_apps` includes `'ORS-3'` |
| `ORS-FC` | `ors_apps` includes `'ORS-FC'` |
| `ORS-R` | `ors_apps` includes `'ORS-R'` (or superadmin) |
| `DB` | `db_role` is any non-empty value |
| `TR` | `db_role` is any non-empty value |
| `MAP` | `db_role` === `'admin'` OR `'agent'` |
| `EVAL` | `eval_access` === true |
| `ADMIN` | `role` === `'admin'` OR `'superadmin'` |

Super admin bypasses all checks. `validUntil` auto-cut applies to all non-super-admin users.

### Real user accounts (land.orsone.app)
- **Balaji** — `balaji@lakshmibalajio2o.com` — superadmin (hardcoded)
- **Ejaz Nathani** — `ejaz_nathani@welspun.com` — Welspun — real developer account
- **Raj Attri** — `raj.attri@ccigroup.co.in` — CCI — real developer account

---

## 8. Key Technical Patterns

### Python3 for multi-line JSX replacements
`sed` fails on multi-line blocks. Always use Python3 inline:
```bash
python3 << 'PYEOF'
with open('src/path/to/file.jsx', 'r') as f:
    c = f.read()
old = """exact multi-line
content to replace"""
new = """replacement
content"""
if old in c:
    c = c.replace(old, new)
    print("OK")
else:
    print("ERROR: string not found")
with open('src/path/to/file.jsx', 'w') as f:
    f.write(c)
PYEOF
```

### Always verify before editing
```bash
grep -n "exact string" src/path/to/file.jsx
# If 0 results: the string is not there — do not proceed
# If 1 result: safe to replace
# If 2+ results: identify which instance before replacing
```

### Always build before pushing
```bash
npm run build
# Must output: ✓ built in X.XXs  (no errors)
# If errors: fix them — do not push broken builds
```

### Git workflow
```bash
git add -A                          # stage all changes
git commit -m "fix/feat: clear description"
git push origin main                # Vercel auto-deploys in ~60s
```

### PAT recovery (if push fails with auth error)
```bash
git remote set-url origin https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git
git push origin main
```

### No Firebase Admin SDK in Vercel
Firebase Admin SDK causes silent 404s via gRPC binding failures in Vercel serverless. Use Google Cloud Storage REST API with `google-auth-library` instead (this applies to ORS-ONE Warehouse `purplebox`, not land platform).

### Circular import protection
`src/modules/Admin/personas.js` exists specifically to break this cycle:
```
Admin/index.jsx  →  imports UserRecord, UserForm
UserRecord.jsx   →  imports PERSONA_META ← must come from personas.js
UserForm.jsx     →  imports PERSONA_META ← must come from personas.js
```
If PERSONA_META were in `Admin/index.jsx`, the TDZ runtime crash returns.

---

## 9. Completed Features (April 2026)

### Public pages
- [x] `HomePage.jsx` — Zoho-inspired scroll reveal, hero, Stage 1/2 explanation, app suite, who it's for, two paths CTA, inline registration form → WhatsApp
- [x] `LandingPage.jsx` — personalised post-login dashboard, time-aware greeting, app tiles by stage, locked apps shown dimmed
- [x] `LoginModal.jsx` — overlay on homepage, show/hide password, forgot password
- [x] `ContactPage.jsx` — full contact details
- [x] `ProductPage.jsx` — per-product info page (508 lines)

### Stage 1 — Transaction Pipeline
- [x] **Deal Board** — demands list, auth gate (admin/agent/client/pending), BCD model, AdminDemandForm with Google Maps Drawing Manager (Circle+Polygon), natural page scroll with sticky sidebars
- [x] **Tracker** — pipeline management, site cards, activity log, WhatsApp share modal
- [x] **Land Map** — 5 road corridors, infrastructure markers, distance calculator, single-row toolbar, Map/Satellite toggle
- [x] **Evaluate** — Zoho landing (stats bar + sidebar filters + card feed), 8-section input form, preview, distance engine, data loading fixed (correct Firebase project), colour theme correct

### Stage 2 — Title Analysis (all Zoho CRM pattern)
- [x] **ORS-1 Ownership** — all views: Dashboard, Families, Members (Family Tree), Land Pedigree, Transfers, Meetings, Site Assets, Digital Assets, Admin
  - Zoho 3-col record views: FamilyRecord, MemberRecord, LandRecord, TransferRecord
  - Site Assets: 4 categories (Site Sketch, Village Map, Location Map, Other), collapsible accordions, migration from legacy digitalAssets
  - Universal Search indexed
- [x] **ORS-2 Reconciliation** — Dashboard, Survey Ledger, Timeline, Gap Analysis, All Records
  - Zoho 3-col record views: IssueRecord (CHK-N codes, Fix-in-Ownership links), SurveyChainRecord
  - Engine: 12 gap check types, survey tree, area accounting, reconciliation score
  - CSV export
- [x] **ORS-3 Succession** — Dashboard, Family Trees, Heir Analysis, Settlement Audit, Mitigation Roadmap
  - Zoho 3-col: HeirRecord (entitlement, flags, dormant)
  - Print Report (combined title analysis, opens new window)
  - Engine: Indian succession law, Class I/II hierarchy, dormant claimant detection
  - CSV + JSON export
- [x] **ORS-FC Field Collect** — mobile data capture
- [x] **ORS-R Title Report** — readiness score, reconciliation score, succession score, risk flags, legal mitigation roadmap, DIY vs Managed delivery models

### Infrastructure
- [x] **Unified colour theme** — entire platform migrated April 2026:
  - Purple `#6141ac` → Blue `#1d9bf0` (91 replacements across 32 files)
  - Lavender background `hsl(259,30%,96%)` → warm off-white `#fafaf8`
  - Header `#1e1537` → steel navy `#1a3a5c`
  - All hardcoded hex values purged from all Stage 2 components
- [x] **AppShell** — universal header, breadcrumb, permission-filtered hamburger menu
- [x] **Universal Search** — Ctrl+K across all 8 apps
- [x] **Title Analysis Disclaimer** — collapsible amber banner in ORS-1/2/3 + print report
- [x] **User Management** — complete Zoho-quality redesign: stat cards, hover-reveal row actions, app-wise access cards with toggles and role selectors, stepped create form with live preview, persona system, validity/expiry system

---

## 10. Known Issues & Critical Warnings

### Evaluate data not loading (debug checklist)
If Evaluate shows 0 reports:
1. Check `src/modules/Evaluate/index.jsx` line 5: must be `import { dbEval } from './firebase'`
2. Check `src/modules/Evaluate/index.jsx` line 5: must **NOT** be `import { db as dbEval } from '../../firebase'`
3. Check that `useEffect([orsProfile])` is present (second effect for async profile load)

### DealBoard scroll magic number
Fixed layers stack: AppShell 52px + header 60px + floating-search 46px = **158px**.  
If scroll breaks, check:
- `.demands-layout { padding-top: 160px }` (2px breathing room)
- `.sidebar { position: sticky; top: 156px; height: calc(100vh - 156px) }`
- Do NOT add `height: calc(100vh - Npx); overflow: hidden` to `.app-body` — this was the old approach that kept breaking

### Circular import (Admin module)
`PERSONA_META` **must** stay in `src/modules/Admin/personas.js`.  
If it is ever moved back into `Admin/index.jsx`, the runtime error `Cannot access 'X' before initialization` (TDZ crash) returns.

### Purple remnant audit
After the April 2026 colour migration, run this to check for any remaining old purple values:
```bash
grep -rn "#6141ac\|hsl(259\|#1e1537\|#1a1030\|rgba(97,65,172" src/ --include="*.jsx" --include="*.js" | grep -v "tokens.js\|global.css"
```
`global.css` intentionally retains old values — it is unused by React components.

### Pending / not started
- [ ] Predictive Analytics — marked "Coming Soon" in ORS-ONE Warehouse (`purplebox`). Gemini 1.5-flash deprecated; pending migration to `gemini-2.0-flash`.
- [ ] AI description generation — same, marked "Coming Soon"
- [ ] Data Governance tools (Transfer Listings, Transfer Leads, Merge Accounts, Deactivate & Reassign, Company Rebrand) — built in `purplebox`, not yet in land platform
- [ ] ORS-FC Field Collect — minimal (32 lines) — full mobile capture form not yet built

---

## 11. Firestore Rules

### `ors-demand-platform` rules
File: `firebase-rules/ors-demand-platform.rules` (committed to repo).  
Must be **manually deployed** to Firebase Console — Vercel does not deploy Firestore rules.  
If Deal Board / Tracker / Land Map show permission-denied errors, the rules file needs redeployment.

### `ors-system-3480b` rules
Standard rules. Super admin has write access. Users can read their own `ors-users` doc. `sites` collection readable by authenticated users with access.

---

## 12. How to Work With This Codebase

### Context: non-developer business owner
- Always explain what you're doing and why in plain English
- State the root cause before proposing a fix
- Never make assumptions — if the issue is unclear, ask before touching code
- No shortcuts, no workarounds, no "this will do for now"
- Consolidated commits — one logical change, one commit

### Standard workflow
```bash
# Clone if container is fresh
git clone https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git /home/claude/ors-platform
cd /home/claude/ors-platform

# 1. Verify target string exists
grep -n "string to find" src/path/to/file.jsx

# 2. Make the edit (Python3 for multi-line, str_replace for simple single-line)
python3 << 'PYEOF'
with open('src/file.jsx','r') as f: c=f.read()
c = c.replace(OLD, NEW)
with open('src/file.jsx','w') as f: f.write(c)
PYEOF

# 3. Build — must succeed before pushing
npm run build
# ✓ built in X.XXs — only then push

# 4. Commit and push
git add -A
git commit -m "fix/feat: clear description of change and root cause"
git push origin main
# → Vercel deploys automatically in ~60 seconds
```

### Working directory
Always use `/home/claude/ors-platform` as the working directory.  
If starting a new session, clone fresh — do not assume the directory exists.

---

*Update this file whenever significant features are completed, architecture changes, or critical bugs are found and fixed.*
