# ORS-ONE Platform — Claude Briefing Document
**Last updated: April 2026 · Maintained in: `LBRBalaji/orsone-home`**

> This file is the single source of truth for any Claude session working on ORS-ONE properties.
> Read this before touching any file. Never make assumptions — always verify with `grep -n` first.

---
> **Note:** The GitHub PAT is redacted in this public document. Balaji has the actual token.
> When starting a session, provide the PAT separately or retrieve it from the private repo.



## 1. What is ORS-ONE?

ORS-ONE is a suite of operational web applications for real estate transactions in India, owned and operated by **Lakshmi Balaji ORS Private Limited**, Chennai.

**Primary tagline:** Building Transaction Ready Assets  
**Secondary tagline:** Transactions Simplified  
**Super admin:** `balaji@lakshmibalajio2o.com` (hardcoded, cannot be revoked, always has full access)  
**Business owner:** Balaji Aram Valartha Sundaram (non-developer, directs all technical development)

The platform is split into two distinct product families:

### Family 1 — Transaction Applications (the "apps")
Public-facing platforms for property sourcing and transactions. Each is a standalone deployment.

| App | URL | Purpose |
|-----|-----|---------|
| Source Warehouse | [lease.orsone.app](https://lease.orsone.app) | India's warehouse leasing marketplace — BCD model |
| Source Land | [land.orsone.app](https://land.orsone.app) | Industrial land: source → site verify → title verify → sellers verify |
| Source Commercial | [source-commercial.orsone.app](https://source-commercial.orsone.app) | Commercial real estate sourcing |
| Residential | [howaah.orsone.app](https://howaah.orsone.app) | Residential property sourcing |
| ORS-ONE Hub | [orsone.app](https://orsone.app) | Public homepage — no login, links to all apps |

### Family 2 — Land Transaction Platform (land.orsone.app)
An integrated 9-module platform built in React/Vite. Two stages:

**Stage 1 — Sourcing & Evaluation**
- Deal Board — demand-specific land sourcing from the market
- Tracker — pipeline management and deal tracking
- Land Map — live map with road corridors and infrastructure overlays
- Evaluate — 8-section preliminary site assessment

**Stage 2 — Title Analysis**
- ORS-1 (Ownership) — families, members, land records, transfer chain
- ORS-2 (Reconciliation) — EC, Patta, survey gap analysis, automated checks
- ORS-3 (Succession) — all heirs mapped, dormant claimants identified
- ORS-FC (Field Collect) — mobile on-site data capture
- ORS-R (Title Report) — transaction readiness score + legal mitigation roadmap

---

## 2. Stack

### land.orsone.app (ors-trace-origin)
| Layer | Technology |
|-------|-----------|
| Framework | React 18 + Vite |
| Language | JavaScript (JSX) — no TypeScript |
| Auth | Firebase Auth (email/password only) |
| Database — main | Firestore `ors-system-3480b` (asia-south1) |
| Database — demand pipeline | Firestore `ors-demand-platform` (Deal Board, Tracker, Land Map) |
| Database — evaluate | Firestore `evaluate-6f1bf` (Evaluate — land_reports collection) |
| Hosting | Vercel — auto-deploys on `git push origin main` (~60s) |
| Repo | `LBRBalaji/ors-trace-origin` (private) |
| GitHub PAT | `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
| Remote URL format | `https://{token}@github.com/LBRBalaji/ors-trace-origin.git` |

### orsone.app (orsone-home)
| Layer | Technology |
|-------|-----------|
| Type | Pure static HTML/CSS/JS — single `index.html` file |
| Hosting | Vercel — auto-deploys on `git push origin main` |
| Repo | `LBRBalaji/orsone-home` (public) |
| GitHub PAT | Same PAT as above |

### lease.orsone.app (ORS-ONE Warehouse)
| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 |
| Database | Firestore `leaseorsone` (asia-south1) |
| Storage | Google Cloud Storage (REST API — not Firebase Admin SDK) |
| AI | Gemini `gemini-2.0-flash` |
| Hosting | Vercel |
| Repo | `LBRBalaji/purplebox` (private) |

---

## 3. Theme & Design System

### land.orsone.app — Zoho-inspired (three strict theme groups, NEVER cross-apply)

**Group 1 — Deal Board + Tracker + Land Map + Evaluate**
```
Font:       DM Sans (body), Space Mono (labels/IDs)
Navy:       #1a3a5c  (header bg, buttons)
Blue:       #1d9bf0  (accent, chip borders, IDs)
Page bg:    #fafal8  (warm off-white)
Muted:      #536471
Border:     #e8e8e4
```

**Group 2 — ORS-1/2/3/FC/Report + Admin (Title Analysis)**
```
Font:       Helvetica Neue / Arial (no monospace)
Primary:    #1d9bf0  (electric blue — was purple #6141ac, migrated April 2026)
Navy:       #1a3a5c  (header, navyText)
Page bg:    #fafaf8  (warm off-white — was lavender hsl(259,30%,96%))
Border:     #e8e8e4
Tokens:     src/shared/tokens.js — single source of truth
```

**Group 3 — Evaluate**
```
Font:       DM Sans (body), Space Mono (labels/IDs)
Navy:       #1a3a5c
Blue:       #1d9bf0
Page bg:    #fafaf8
```

**Single source of truth:** `src/shared/tokens.js` — import `T` from there. Never hardcode hex values in components.

**AppShell constants** (`src/components/AppShell.jsx`):
```js
const NAVY   = '#1a3a5c'
const PURPLE = '#1d9bf0'   // renamed PURPLE but value is blue
const PURPLEM= 'rgba(29,155,240,0.65)'
```

### orsone.app — LynkGrids-inspired (single colour, editorial)
```
Accent:     #0055FF  (one blue, used everywhere — NO other accent colours)
Ink:        #111111
Mid:        #555555
Faint:      #888888
Rule:       #E4E4E0
Surface:    #F7F7F5
Background: #FFFFFF
```
No multi-colour sections. No cinematic shapes. Section separation by 1px border lines only.

---

## 4. Business Model

**BCD Model:** Broker · Client · Developer — all three on one platform.

**Two access paths:**
- **Option A — DIY:** Platform access fee. User manages everything.
- **Option B — Managed Service:** ORS team handles data entry, document procurement, full analysis and delivers the final Transaction Report.

**Source · Engage · Transact** — the three-step journey across all asset classes.

**lease.orsone.app specific:**
- Built on 20+ years domain expertise, 3.5M+ sq ft transacted across Tamil Nadu
- Previously built Followprop (8,000+ warehouses aggregated) — that taught the market wants a marketplace, not a database
- **Instant Download** is the #1 feature requirement from 2010–2024 market study
- Verified listings only — reviewed and approved before going live

---

## 5. User Roles

### land.orsone.app roles (Firestore: `ors-users/{uid}`)

| Role | Description |
|------|-------------|
| `superadmin` | `balaji@lakshmibalajio2o.com` — hardcoded, full access to everything, cannot be revoked |
| `admin` | Platform administrators |
| `user` | Standard users with granular per-app permissions |

**Per-user permission fields:**
```js
{
  email:               string,
  displayName:         string,
  persona:             'internal' | 'partner' | 'aggregator' | 'service' | 'client' | 'other',
  db_role:             '' | 'admin' | 'agent' | 'client' | 'viewer',   // Deal Board role
  eval_access:         boolean,      // Evaluate access
  ors_apps:            string[],     // ['ORS-1','ORS-2','ORS-3','ORS-FC','ORS-R']
  ors_projects:        string[],     // site IDs — empty = all sites
  validUntil:          string|null,  // ISO date — access auto-cut on expiry
  validityReminderDays: number,      // days before expiry to show warning (default 14)
  active:              boolean,
  approvedBy:          string,       // email of approver
  approvedAt:          string,       // ISO date
  company:             string,
  phone:               string,
  notes:               string,
}
```

**Land Map access:** Derived from `db_role` — enabled automatically for `admin` and `agent` roles.

### Persona types
| Key | Label |
|-----|-------|
| `internal` | Internal Team |
| `partner` | Operating Partner |
| `aggregator` | Team Aggregator |
| `service` | Service Provider |
| `client` | Client |
| `other` | Other |

### Deal Board roles (land.orsone.app)
| Role | Capabilities |
|------|-------------|
| `admin` | Full access — post demands, manage all, access Land Map |
| `agent` | Submit sites, access Land Map |
| `client` | View demands, download, submit matching property |
| `viewer` | Read-only |
| `''` | No access |

### Real developer accounts (land.orsone.app)
- Ejaz Nathani — `ejaz_nathani@welspun.com` (Welspun)
- Raj Attri — `raj.attri@ccigroup.co.in` (CCI)

---

## 6. Key Rules

### Code rules
1. **Always `grep -n` before editing** — verify the exact string exists in the exact file before any replacement
2. **Python3 inline scripts for multi-line replacements** — `sed` fails on multi-line JSX blocks. Use:
   ```bash
   python3 << 'PYEOF'
   with open('src/file.jsx', 'r') as f: c = f.read()
   c = c.replace(OLD, NEW)
   with open('src/file.jsx', 'w') as f: f.write(c)
   PYEOF
   ```
3. **No Firebase Admin SDK in Vercel API routes** — causes silent 404s via gRPC binding failures. Use Google Cloud Storage REST API with `google-auth-library` instead.
4. **`serverActions.bodySizeLimit` does not apply to API route handlers** in Next.js — only applies to Server Actions.
5. **Never use `--break-system-packages` on Vercel** — only on local Ubuntu containers.
6. **Always build before pushing:** `npm run build` must succeed with `✓ built` before `git push`.
7. **Consolidated commits** — one commit per logical fix, not one per file.
8. **No risky regex on JSX** — always Python string replace, never sed on multi-line blocks.

### Design rules (land.orsone.app)
1. **Never cross-apply themes between the three groups** — Evaluate theme ≠ Title Analysis theme
2. **No hardcoded hex values in components** — always use `T.purple`, `T.navy` etc. from tokens.js
3. **No emojis in UI** — never
4. **No "O in a square box" logo** — never
5. **Header is single line:** ORS + "Ownership · Reconciliation · Succession" + module badge + breadcrumb
6. **Footer:** "Lakshmi Balaji ORS Private Limited" left, "Building Transaction Ready Assets" centre
7. **Zoho CRM pattern** (not Zoho CRM — the Zoho homepage aesthetic): filter tabs → table → row-click → 3-col record view
8. **Root cause fixes only** — no workarounds, no band-aids
9. **No repeated work on a single task** — get it right once

### Design rules (orsone.app)
1. **One accent colour only:** `#0055FF` — nothing else is coloured
2. **No cinematic multi-colour sections** — sections separated by 1px border lines only
3. **No icons** — typography and layout carry the design
4. **Mobile first** — hamburger drawer, all grids collapse, 44px touch targets minimum
5. **LynkGrids-inspired** — pure white, editorial, whitespace-led

---

## 7. Architecture

### land.orsone.app routing (App.jsx)
```
/  (root) →
  Loading → auth checking
  No user → HomePage.jsx (pre-login, public marketing page)
  User authenticated → LandingPage.jsx (personalised dashboard, app tiles)
  User selects app → navigate(moduleId)
    'ORS-1' → SiteWorkspace (initialModule='ORS-1')
    'ORS-2' → SiteWorkspace (initialModule='ORS-2')
    'ORS-3' → SiteWorkspace (initialModule='ORS-3')
    'ORS-FC'→ SiteWorkspace (initialModule='ORS-FC')
    'ORS-R' → SiteWorkspace (initialModule='ORS-R')
    'DB'    → DealBoard
    'TR'    → Tracker (iframe to original HTML)
    'MAP'   → LandMap (iframe to original HTML)
    'EVAL'  → Evaluate
    'ADMIN' → Admin (super admin only)
```

### SiteWorkspace
All Title Analysis modules route through `SiteWorkspace/index.jsx`. It renders:
- A site selector (ProjectSelector)
- Once a site is selected, the appropriate ORS module view
- ORS-1 through ORS-FC are full React rewrites
- The ORS-R (Title Report) is also React

### Three Firebase projects
```
ors-system-3480b     → Main auth + ors-users + sites + all ORS1/2/3/FC data
ors-demand-platform  → Deal Board (demands) + Tracker + Land Map
evaluate-6f1bf       → Evaluate (land_reports collection)
```

**Critical:** The Evaluate module (`src/modules/Evaluate/`) has its own `firebase.js` that initialises `evaluate-6f1bf`. Always import `dbEval` from `./firebase`, NOT from `../../firebase`.

### Circular import danger (Admin module)
`PERSONA_META` is in `src/modules/Admin/personas.js` — a standalone file. It was extracted to break a TDZ circular import:
- `Admin/index.jsx` imports `UserRecord` and `UserForm`
- Both of those previously imported `PERSONA_META` from `Admin/index.jsx`
- This caused `Cannot access 'X' before initialization` at runtime
- **Never move `PERSONA_META` back into `Admin/index.jsx`**

---

## 8. Key Files

### land.orsone.app (`LBRBalaji/ors-trace-origin`)

| File | Purpose |
|------|---------|
| `src/App.jsx` | Root routing — auth state → page dispatch |
| `src/firebase.js` | Main Firebase config (`ors-system-3480b`) |
| `src/hooks/useAuth.js` | Auth hook — `SUPER_ADMIN`, `canAccess()`, `canAccessProject()` |
| `src/shared/tokens.js` | **Design tokens — single source of truth for all colours** |
| `src/shared/ui.jsx` | Shared UI components: `Btn`, `Toast`, `ConfirmDialog`, `SearchInput` |
| `src/shared/ZohoTable.jsx` | Reusable Zoho-style table component |
| `src/components/AppShell.jsx` | Universal header (NAVY/PURPLE/PURPLEM constants live here) |
| `src/components/LoginModal.jsx` | Login overlay — navy/blue theme, show/hide password, forgot password |
| `src/pages/HomePage.jsx` | Pre-login public page — Zoho-inspired, scroll reveal |
| `src/pages/LandingPage.jsx` | Post-login personalised dashboard — app tiles by stage |
| `src/modules/Admin/index.jsx` | User Management list — stat cards, search, rich table |
| `src/modules/Admin/views/UserRecord.jsx` | User detail — 3-col, app-wise access cards, toggle switches |
| `src/modules/Admin/views/UserForm.jsx` | Create user — stepped form, live access preview |
| `src/modules/Admin/personas.js` | `PERSONA_META` — extracted to break circular import |
| `src/modules/DealBoard/` | Deal Board — demands, BCD model, Google Maps drawing |
| `src/modules/DealBoard/views/DemandsListView.jsx` | Natural page scroll, sticky sidebars at top:156px |
| `src/modules/DealBoard/styles.js` | All Deal Board CSS as JS template literal |
| `src/modules/LandMap/index.jsx` | Land Map with toolbar (single row: search + type + category) |
| `src/modules/Evaluate/index.jsx` | Evaluate — Zoho landing, form, preview |
| `src/modules/Evaluate/firebase.js` | **Evaluate's own Firebase config (`evaluate-6f1bf`)** |
| `src/modules/SiteWorkspace/index.jsx` | Title Analysis router — site selector + module views |
| `src/modules/ORS1/ProjectSelector.jsx` | Site cards landing (Stage 2 entry point) |
| `src/modules/ORS1/views/` | Families, Members, Land Records, Transfers, Site Assets, Digital Assets, Meetings, Pedigree |
| `src/modules/ORS2/views/` | Overview, Survey Ledger, Gap Analysis, Issue Record, Survey Chain Record |
| `src/modules/ORS3/views/` | Overview, Heir Analysis, Family Trees, Settlement Audit, Print Report |
| `src/modules/ORSR/index.jsx` | Title Report module |

### orsone.app (`LBRBalaji/orsone-home`)

| File | Purpose |
|------|---------|
| `index.html` | Entire homepage — single static file, all CSS/JS inline |
| `vercel.json` | `cleanUrls: true, trailingSlash: false` |
| `BRIEFING.md` | This file |

---

## 9. Completed Features (as of April 2026)

### orsone.app
- [x] Full homepage — LynkGrids-inspired design, single `#0055FF` accent, no multi-colour
- [x] Correct warehouse content — BCD model, 20yr/3.5M sq ft credibility, Instant Download story
- [x] land.orsone.app section — 4-step journey (Source → Site Verified → Title Verified → Sellers Verified)
- [x] Commercial and Residential app sections
- [x] Tools grid (9 tools), Knowledge Hub, YouTube cards
- [x] For Who section (Clients / Developers / Brokers)
- [x] Two Paths section (DIY vs Managed Service)
- [x] Enquiry form with role and interest selectors
- [x] Footer — brand, contact, app links, tools links
- [x] Mobile responsive — hamburger drawer, all grids collapse at 960px and 600px
- [x] Vercel deployment via `LBRBalaji/orsone-home` GitHub repo

### land.orsone.app — Stage 1
- [x] Deal Board — live demands, BCD model, Google Maps Drawing Manager, auth gate
- [x] Tracker — pipeline management
- [x] Land Map — live map, road corridors, distance calculator, toolbar single row
- [x] Evaluate — Zoho landing (stats bar + sidebar + card feed), form, preview, data loading fixed

### land.orsone.app — Stage 2 (Title Analysis)
- [x] ORS-1 Ownership — Families, Members, Land Records, Transfers, Site Assets, Digital Assets, Meetings, Pedigree — all Zoho CRM pattern
- [x] ORS-2 Reconciliation — Overview, Survey Ledger, Gap Analysis, Issue Record, Survey Chain Record — all Zoho CRM
- [x] ORS-3 Succession — Overview, Heir Analysis, Family Trees, Settlement Audit, Print Report — all Zoho CRM
- [x] ORS-FC Field Collect — data capture
- [x] ORS-R Title Report

### Admin / User Management
- [x] List view — stat cards (All/Active/Pending/Expiring/Revoked), search, rich table with hover actions
- [x] User Record — 3-col layout, app-wise access cards per module, smooth toggles, site access chips
- [x] Create User form — stepped layout, live Access Preview panel
- [x] Validity / expiry system — auto-lock on expiry date, warning banner N days before
- [x] Persona system (6 types: internal, partner, aggregator, service, client, other)
- [x] Circular import TDZ bug fixed — `PERSONA_META` extracted to `personas.js`

### Infrastructure
- [x] Unified colour theme — Stage 2 migrated from purple `#6141ac` to blue `#1d9bf0` (91 replacements across 32 files)
- [x] All hardcoded hex values purged from Stage 2 components
- [x] Pre-login homepage — scroll reveal, hero, stats, how it works, app suite, who it's for, two paths CTA
- [x] Post-login dashboard — personalised greeting, accessible app tiles grouped by stage, locked apps shown dimmed
- [x] Login modal — unified blue/navy theme, show/hide password, forgot password flow
- [x] AppShell header — NAVY `#1a3a5c`, breadcrumb, module badge

---

## 10. Known Issues / Pending

### Active issues
- [ ] **Hardcoded purple remnants** — some edge-case components in ORS1/ORS2/ORS3 may still have inline purple values not caught in the April 2026 sweep. Run `grep -rn "#6141ac\|hsl(259" src/` to audit.
- [ ] **Evaluate data loading** — auth timing fix applied (second `useEffect` on `orsProfile`), but if user sees 0 reports, check: (1) is `dbEval` importing from `./firebase` not `../../firebase`? (2) is `orsProfile` loading before the auth listener fires?
- [ ] **DealBoard scroll** — switched to natural page scroll (sticky sidebars at `top:156px`). If scroll issues return, the magic number is: AppShell 52px + DealBoard header 60px + floating search 46px = 158px. Adjust `top` and `padding-top` on `.demands-layout`.

### Pending work (not started)
- [ ] Predictive Analytics — marked "Coming Soon" (Gemini 1.5-flash deprecated, pending migration to gemini-2.0-flash)
- [ ] AI description generation — same, marked "Coming Soon"
- [ ] Data Governance tools (Manage Users: Transfer Listings, Transfer Leads, Merge Accounts, Deactivate & Reassign, Company Rebrand) — built in ORS-ONE Warehouse (`purplebox`), not yet in land platform
- [ ] orsone.app GoDaddy DNS — A record needs pointing to Vercel project for `orsone.app`. Currently may still point to old Deal Board deployment.
- [ ] source-commercial.orsone.app and howaah.orsone.app — listed on orsone.app but content/status not confirmed
- [ ] Community platform (`lease.orsone.app/community`) — live and functional but not integrated with land platform

---

## 11. How to Work With This Codebase

### You are working with a non-developer business owner
- Explain what you are doing in plain English before and after every change
- State the root cause, not just the fix
- Never make assumptions — if something is unclear, ask before touching code
- No shortcuts, no workarounds, no "this will do for now"

### The workflow
```bash
# 1. Always clone fresh if the container is new
git clone https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git /home/claude/ors-platform
cd /home/claude/ors-platform

# 2. Always verify the target string exists before replacing
grep -n "exact string to find" src/path/to/file.jsx

# 3. For simple single-line changes — str_replace or direct edit
# For complex multi-line JSX replacements — Python3 inline script:
python3 << 'PYEOF'
with open('src/path/to/file.jsx', 'r') as f:
    c = f.read()
old = """exact multi-line
old content here"""
new = """replacement
content here"""
if old in c:
    c = c.replace(old, new)
    print("OK")
else:
    print("ERROR: not found")
with open('src/path/to/file.jsx', 'w') as f:
    f.write(c)
PYEOF

# 4. Always build before pushing
npm run build
# Must output: ✓ built in X.XXs
# If errors: fix them before pushing

# 5. Commit and push
git add -A
git commit -m "fix/feat: clear description of what changed and why"
git push origin main
# Vercel auto-deploys in ~60 seconds after push

# 6. Verify in Vercel before telling user it's live
```

### For orsone.app (static HTML)
```bash
cd /home/claude/orsone-home
# Edit index.html directly — no build step needed
# Verify by reading the file: grep -n "search term" index.html
git add -A
git commit -m "description"
git push origin main
# Vercel deploys in ~30 seconds
```

### PAT (GitHub Personal Access Token)
```
ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
Used for both `ors-trace-origin` and `orsone-home` repos.
Remote URL format: `https://{PAT}@github.com/LBRBalaji/{repo}.git`

If push fails with auth error:
```bash
git remote set-url origin https://ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/LBRBalaji/ors-trace-origin.git
git push origin main
```

---

## 12. Conversation History Summary

This platform has been built incrementally over many sessions. Key milestones:

| Period | Work done |
|--------|-----------|
| Early sessions | Initial ORS-1 (Ownership) build — Families, Members, Land Records, Transfers |
| Phase 2 | Zoho CRM pattern applied — ZohoTable, 3-col record views, filter tabs |
| Phase 3 | ORS-2 Reconciliation redesigned — Survey Ledger, Gap Analysis, Issue Record |
| Phase 4 | ORS-3 Succession redesigned — Heir Analysis, Family Trees, Print Report |
| DealBoard | Auth gate restored, Google Maps Drawing Manager, BCD model, scroll fixed |
| User Management | Full Zoho-quality UX — stat cards, rich table, app-wise access cards, create form with live preview |
| Colour migration | Entire Stage 2 migrated from purple `#6141ac` → blue `#1d9bf0`, lavender bg → warm off-white `#fafaf8`. 91 replacements across 32 files. AppShell and tokens.js updated. |
| Evaluate | Data loading fixed (wrong Firebase project imported), Zoho landing redesigned, correct colour theme |
| orsone.app | Homepage built — first version, then warehouse content corrected from live site, mobile responsiveness added, then full redesign to LynkGrids single-colour editorial style |
| LandMap | Toolbar fixed to single row (search + type + category) |
| Login | Modal redesigned to blue/navy unified theme |
| Post-login | LandingPage replaced with personalised dashboard — greeting + accessible app tiles |
| Admin | Complete UX rebuild — stat header, hover-reveal row actions, UserRecord with app cards, UserForm with stepped layout and live preview |

---

*End of briefing. Update this file whenever significant features are completed or architecture changes.*
