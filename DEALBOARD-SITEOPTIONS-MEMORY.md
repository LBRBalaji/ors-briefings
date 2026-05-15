# haanest — Deal Board & Site Options: Technical Memory
## Logic, Architecture, Data Flow, Algorithms

---

## 1. Firebase Projects & Collections

Two separate Firebase projects are used:

| Project | Usage |
|---|---|
| `ors-demand-platform` | demands, sites, demand-sites, propertySubmissions, activity, publicStats, site-validations |
| `ors-system-3480b` | ors-users (auth profiles, access control) |

Both are initialised as separate Firestore `db` instances. The DealBoard/Tracker modules import from `./firebase` which points to `ors-demand-platform`.

---

## 2. Demand Creation (Admin)

### Entry Point
`AdminDemandForm.jsx` — rendered when admin clicks "Add Demand" or "✎ Edit" on an existing card.

### Demand ID Algorithm
```js
function genDemandId() {
  const n   = new Date()
  const pad = x => String(x).padStart(2, '0')
  // Format: D-YYYYMMDD-NN (e.g. D-20260227-18)
  return `D-${n.getFullYear()}${pad(n.getMonth()+1)}${pad(n.getDate())}-${String(Math.floor(Math.random()*99)+1).padStart(2,'0')}`
}
```
Two-digit random suffix prevents collisions on same-day creation. No sequential counter — fully client-generated.

### Form Fields Saved to Firestore (`demands` collection)
```
type, location, area, budget, transaction, details,
preferredAreas (freetext), mapShape (JSON string),
status ('active'|'hot'|'new'|'archived'),
clientId, clientName,
proposals (counter, starts at 0),
refCode, assignedUsers (array of emails),
demandId, date (display string), createdAt (serverTimestamp)
```

### Map Shape Storage
The drawing tool (Google Maps `DrawingManager`) captures either a Circle or Polygon. Before saving to Firestore, it is serialised:
```js
// Circle
{ type: 'circle', lat: 12.34, lng: 78.56, radius: 5000 }

// Polygon
{ type: 'polygon', path: [{lat, lng}, {lat, lng}, ...] }
```
Stored as `JSON.stringify(shape)` — Firestore has no geometry type. On display, `JSON.parse()` reconstructs it. The `DemandMap` component reads the type field and renders either `google.maps.Circle` or `google.maps.Polygon`.

### Create vs Update Logic
```js
if (editDemand) {
  await updateDoc(doc(db, 'demands', editDemand.id), { ...fields, updatedAt: serverTimestamp() })
} else {
  await addDoc(collection(db, 'demands'), { ...fields, demandId: genDemandId(), createdAt: serverTimestamp() })
}
```

---

## 3. Deal Board — Demand Display

### Data Loading Strategy: Two-Phase
```js
// Phase 1: getDocs (immediate, single fetch)
const snap = await getDocs(query(collection(db,'demands'), orderBy('createdAt','desc')))
setDemands(snap.docs.map(d => ({id: d.id, ...d.data()})))

// Phase 2: onSnapshot (real-time listener)
const unsub = onSnapshot(q, snap => { setDemands(...) })
return () => unsub()  // cleanup on unmount
```
Phase 1 gives instant render. Phase 2 keeps it live — any admin edit/archive/new demand updates all open browser sessions simultaneously without refresh.

### Filtering Algorithm
Client-side only — no Firestore query filters (avoids composite index requirements):
```js
const filtered = demands.filter(d => {
  if (d.status === 'archived') return false   // archived hidden by default
  if (locFilter  && d.location !== locFilter) return false
  if (typeFilter && d.type     !== typeFilter) return false
  if (search) return JSON.stringify(d).toLowerCase().includes(search.toLowerCase())
  return true
})
```
`JSON.stringify(d)` search is intentional — searches all fields including nested objects in one pass.

### Archived Demands
Toggle state `showArchived` inverts the filter:
```js
if (showArchived  && !isArchived) return false
if (!showArchived &&  isArchived) return false
```
Archived docs remain in Firestore with `status: 'archived'`. Admin restores with `updateDoc → status: 'active'`. Permanent delete uses `deleteDoc`.

### Public Access
`DealBoard` renders without auth for browsing. The "Submit Matching Property" button checks `isLoggedIn` before opening `PropertySubmitModal`. If not logged in, `pendingSubmit` stores `{ demandId, demandDocId }` and the auth flow completes, then replays the submit.

---

## 4. Property Submission Flow

### Entry Point
Partner/Agent clicks "Submit Matching Property" on any demand card → `PropertySubmitModal`.

### What handleSubmit Does (3 parallel writes)

```
Step 1: propertySubmissions collection
  → Full submission record with status: 'pending'
  → assignments: [], isAssigned: false
  → submittedBy: { uid, name, email, role }

Step 2: sites collection (all-sites)
  → Mirrors relevant fields from submission
  → siteId generated: `LB-${DDMMYY}-${docId.slice(-5).toUpperCase()}`
  → Written as draft, then siteId patched via updateDoc

Step 3: demand-sites collection (if linked to demand)
  → Exact copy of sites doc
  → Extra fields: mirrorOf: siteRef.id, linkedDemandId: demandId
```

### Site ID Algorithm
```js
const ddmmyy = new Date().toLocaleDateString('en-GB', {
  day:'2-digit', month:'2-digit', year:'2-digit'
}).replace(/\//g,'')
// e.g. "140526"

const siteId = `LB-${ddmmyy}-${docRef.id.slice(-5).toUpperCase()}`
// e.g. "LB-140526-JVV4U"
```
Last 5 chars of Firestore's auto-generated doc ID (base62 random). Combined with date prefix gives human-readable, collision-resistant ID. Always uppercase.

### proposals Counter
After submission, the demand's `proposals` field is incremented:
```js
await updateDoc(doc(db,'demands', demandDocId), {
  proposals: increment(1)   // Firestore atomic increment
})
```
This is why the demand card shows `12 proposals` without reading all submissions.

---

## 5. Site Options (Tracker) — Data Architecture

### Three Data Collections

```
sites              → master record for every site (all agents, all admins)
demand-sites       → mirror of sites that are linked to a specific demand
propertySubmissions → raw submission from Deal Board (agent's perspective)
```

### Mirror Pattern (Critical Logic)
Every site that is linked to a demand exists in BOTH `sites` AND `demand-sites`. This is intentional:

- `sites` = admin's master inventory (All Sites tab)
- `demand-sites` = demand-specific view (Demand Specific Sites tab, LandMap filter)

When a site is edited, BOTH docs must be updated. The link is:
```
sites/{docId}          ← sourceDocId / canonical record
demand-sites/{docId}   ← mirrorOf: sites.docId, linkedDemandId: demandId
```

### Source Document ID Resolution
Because demand-sites docs have a different Firestore ID from their sites counterpart, any write must resolve the correct target:
```js
const targetId = site.sourceDocId || site.mirrorOf || site.id
await updateDoc(doc(db, 'sites', targetId), fields)        // update canonical
await updateDoc(doc(db, 'demand-sites', site.id), fields)  // update mirror
```
`site.id` used for demand-sites write (that's the mirror doc's own ID).
`targetId` used for sites write (the canonical doc's ID).

### Loading Strategy
```js
async function loadAdminSites() {
  // All sites — ordered by timestamp desc
  const snap = await getDocs(query(collection(db,'sites'), orderBy('timestamp','desc')))
  setAllSites(snap.docs.map(d => ({id:d.id, ...d.data()})))

  // All demands — for dropdowns and linking
  const dSnap = await getDocs(query(collection(db,'demands'), orderBy('createdAt','desc')))
  setDemands(dSnap.docs.map(d => ({id:d.id, ...d.data()})))

  // Demand-specific sites — separate collection
  const dsSnap = await getDocs(query(collection(db,'demand-sites'), orderBy('timestamp','desc')))
  setDemandSites(dsSnap.docs.map(d => ({id:d.id, ...d.data()})))
}
```
Note: `getDocs` not `onSnapshot` here — Tracker does not need real-time updates for site inventory. Manual "↺ Refresh" button triggers reload.

### Role-Based Loading
```js
if (isAdmin || canViewAllSites) { loadAdminSites(); loadAgents() }
else if (isClient && assignedDemandId) { loadAdminSites() }  // load to filter
else loadAgentSites()  // agent sees only their own submissions
```

Agent loading filters by `uid`:
```js
getDocs(query(collection(db,'sites'), where('uid','==',user.uid), orderBy('timestamp','desc')))
```

---

## 6. Demand Specific Sites Tab

### Grouping & Sorting Algorithm
Sites are grouped into three buckets based on `_feasible` field (a local React state field, not stored in Firestore — populated from `site-validations` collection on load):

```js
const feasibleSites    = visibleDs.filter(s => s._feasible === 'Feasible')
const notFeasibleSites = visibleDs.filter(s => s._feasible === 'Not Feasible')
const pendingSites     = visibleDs.filter(s => !s._feasible || s._feasible === '')

// Render order: Feasible → Pending → Not Feasible
const groups = [
  { key:'feasible',    sites: feasibleSites    },
  { key:'pending',     sites: pendingSites     },
  { key:'notfeasible', sites: notFeasibleSites },
].filter(g => g.sites.length > 0)
```

`_feasible` is a computed local field. It is populated when `SiteValidationReport` loads from `site-validations` and calls `onFeasibilityChange(feasibility)` → `setDemandSites(prev => prev.map(x => x.id === s.id ? {...x, _feasible: feasibility} : x))`. This means the sort order updates live as validation reports load, without a page refresh.

### Demand Filter (Dropdown)
Only demands that have at least one linked site appear in the dropdown:
```js
demands.filter(d =>
  d.demandId &&
  demandSites.some(s => s.linkedDemandId === d.demandId || s.demandId === d.demandId)
)
```

### Client Access Control
Client users (`db_role === 'client'`) have `assignedDemandId` on their profile. They see only demand-specific sites tab, filtered to their demand:
```js
const clientFilter = isClient && assignedDemandId ? assignedDemandId : selDemand
const visibleDs = clientFilter
  ? demandSites.filter(s => s.linkedDemandId === clientFilter || s.demandId === clientFilter)
  : demandSites
```

---

## 7. Site Validation Report

### Storage
Separate collection `site-validations` in `ors-demand-platform`, keyed by `siteId` (the human-readable LB-DDMMYY-XXXXX ID):
```js
setDoc(doc(db, 'site-validations', siteId), {
  feasible, saleType,
  extentWilling, portionWilling,
  truckAccess, operationalFeasibility,
  saleTerms, challenges, risks,
  comments: [],           // arrayUnion for appending
  siteId, linkedDemandId,
  updatedAt, updatedBy
}, { merge: true })
```

`merge: true` — safe to call multiple times, only updates provided fields.

Comments use Firestore `arrayUnion`:
```js
await updateDoc(doc(db,'site-validations', docId), {
  comments: arrayUnion({ text, by: userEmail, at: new Date().toISOString() })
})
```
This is atomic — concurrent comments don't overwrite each other.

### Load and Feasibility Signal
```js
useEffect(() => {
  getDoc(doc(db,'site-validations', docId)).then(snap => {
    const data = snap.exists() ? snap.data() : EMPTY
    setReport(data)
    // Signal parent immediately so summary bar counts update
    if (data.feasible && onFeasibilityChange) onFeasibilityChange(data.feasible)
  })
}, [docId])
```

---

## 8. Activity Log

Every site addition is logged to `activity` collection:
```js
await addDoc(collection(db,'activity'), {
  type: 'new_site',
  siteId, village, acres, landType,
  uid, agentEmail, agentName,
  demandId: demandId || null,
  timestamp: serverTimestamp()
})
```
Loaded in Tracker's Activity tab ordered by timestamp. Not real-time — loaded on tab click.

---

## 9. Access Control Layer

### canAccess Function (useAuth.js)
Checks profile from `ors-system-3480b / ors-users / {uid}`:
```js
function canAccess(mod) {
  if (!profile || !profile.active) return false

  // Time-bound access
  if (profile.validUntil) {
    const exp = new Date(profile.validUntil)
    if (!isNaN(exp.getTime()) && exp < new Date()) return false
  }

  // Per-URL granular roles (set in /admin User Management)
  if (mod === 'DB')  return !!(profile.db_role) || profile.db_access  !== 'none'
  if (mod === 'TR')  return !!(profile.tr_role)  || profile.tr_access  !== 'none'
  if (mod === 'MAP') return !!(profile.map_role) || profile.map_access !== 'none' || !!(profile.db_role)
  if (mod === 'EVAL') return !!profile.eval_access

  // ORS Title Analysis modules
  if (['ORS-1','ORS-2','ORS-3','ORS-FC'].includes(mod))
    return (profile.ors_apps||[]).includes(mod)
}
```

### isAdmin in DealBoard Context
DealBoard has its own `useDBAuth` hook separate from `useAuth`:
```js
const isAdmin = isSA || profile?.db_role === 'admin'
```
Where `isSA` = `user.email === DB_ADMIN` (hardcoded superadmin email). This means admin status in DealBoard is determined by `ors-demand-platform` profile's `db_role`, not the `ors-system-3480b` profile.

---

## 10. User Profile Fields (ors-system-3480b / ors-users)

```
uid              — Firestore doc ID = Firebase Auth uid
email            — lowercase
displayName      — full name
company          — organisation
phone
role             — 'admin' | 'user'
status           — 'active' | 'pending' | 'hold' | 'frozen'
active           — boolean (quick access gate)
userType         — identity label (admin/agent/customer/demand-assistant/etc.)
db_role          — Deal Board role (legacy, broad)
db_access        — granular: Deal Board role
tr_role / tr_access  — Site Options role
map_role / map_access — Sites Map role
eval_access      — boolean: Evaluate app
svr_access       — boolean: SVR field app
platform_manager — boolean: admin panel
ors_apps         — array: ['ORS-1','ORS-2','ORS-3','ORS-FC','ORS-R']
assignedDemandId — demand ID for client-restricted access
validUntil       — ISO date string, access auto-expires
firstLogin       — boolean: triggers Terms modal
termsAccepted    — boolean
createdAt / updatedAt
```

---

## 11. New User Creation Flow

1. Admin creates user at `/admin` → `UserForm` → `onSave(data)`
2. Firestore doc written under `pending-{email}` key (admin cannot create Auth accounts client-side — `ors-system-3480b` rules require `request.auth.uid == uid` for creates)
3. `sendSignInLinkToEmail` or `sendPasswordResetEmail` sent to user's email
4. User clicks email link → Firebase Auth creates their account → `onAuthStateChanged` fires
5. `useAuth` detects `pending-` prefixed doc by email → migrates it to `ors-users/{real-uid}` → deletes pending doc
6. `firstLogin: true` triggers Terms of Use modal → on accept → `ChangePasswordModal`

---

## 12. Key Constraints for Howaah / Other Apps

- **Never store Maps SDK objects in Firestore** — always `JSON.stringify` shapes first
- **Mirror pattern** — if you need demand-specific filtering, maintain two collections (all-items + demand-items) with explicit mirror writes on every update
- **Atomic proposal counter** — use Firestore `increment()` not read-then-write
- **Two-phase loading** — `getDocs` for instant paint, `onSnapshot` for live updates
- **siteId formula** — `${prefix}-${DDMMYY}-${docId.slice(-5).toUpperCase()}` is human-readable and collision-resistant without a sequence counter
- **Source doc resolution** — always `site.sourceDocId || site.mirrorOf || site.id` before any write; `site.id` alone will fail when editing mirror docs
- **arrayUnion for comments** — concurrent writes are safe; never read-modify-write an array in Firestore
- **canAccess time-bound** — check `validUntil` before any role check; expired profiles get `false` on all modules
