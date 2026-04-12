# TorqueClub Members Portal — CLAUDE.md

## Project Overview

**TorqueClub** is a full-featured club management portal for classic car clubs. The live demo instance is configured as the **Classic Motoring Club of Australia (CMCA)** and deployed at:

- **Live demo:** https://torqueclub-demo.netlify.app
- **GitHub repo:** https://github.com/pa747sp/RCCA.git (`demo` branch)
- **Auto-deploy:** Netlify watches the `demo` branch — push to deploy (no build step)

---

## Architecture

### Single-file app

The entire application is a **single `index.html` file** (~15,960 lines, ~1.1 MB). There is no build pipeline, no bundler, no npm dependencies beyond CDN links.

```
index.html
  └── React 18 (CDN)
  └── Babel Standalone (CDN — processes JSX in-browser)
  └── 3 × <script type="text/babel"> blocks
```

### The three script blocks

Babel Standalone processes each `<script type="text/babel">` block independently. All three must parse successfully or nothing renders.

| Block | Approx. lines | Contents |
|-------|--------------|----------|
| Block 1 | 29–4921 | `CLUB_CONFIG`, global CSS, seed data, UI primitives, all member-facing sections (Members, Vehicles, Meetings, Events, Dues, Tasks, Incidents, Polls, Marketplace, Shop, Profile, Dashboard, Login, Public pages) |
| Block 2 | 4923–10761 | Finance, Applications, Website/Newsletter, Records, Recognition, Points, Library, App, `getCats()` |
| Block 3 | 10763–end | Parts Inventory, Parts Catalogue, Commerce wrapper, Vehicles wrapper, Contacts, Suppliers, Competitions, Records wrapper, Quality System, Archive, Points & Awards, Permissions, Account Section, Administration Section, Gallery, Event Workflow |

**Critical rule:** JSX requires all tags to be explicitly closed — a missing `</tr>` before `</tfoot>` will silently blank the entire page. Always validate with Babel after edits (see Testing section below).

### Global state pattern

- All state lives in `App()` (Block 2, ~line 8480)
- State is collected into a `sp` (spread props) object and passed to section components via `{...sp}`
- Storage: `window.storage` → `localStorage` wrapper with `dbGet(key)` / `dbSet(key, value)` async helpers
- Storage keys: `SK` object, all keyed as `CLUB_CONFIG.clubId + ":keyname"` (clubId = `"demo"`)
- New data: add key to `SK`, add `useState`, add to `Promise.all` load, add to `sp`

### CLUB_CONFIG

The `CLUB_CONFIG` object (Block 1, ~line 31) holds runtime club identity. It is a mutable `const` — properties can be updated in-place for live propagation without page reload. The Account section does this on save.

### CSS variables

Theme colours live in `:root` CSS variables. Key ones:
- `--burg` — primary accent (buttons, active states, highlights)
- `--burg2/3/dk/xdk` — primary colour variants
- `--gold`, `--gold2`, `--gold3` — secondary accent / headings
- `--bg`, `--bg2`, `--bg3`, `--bg4` — background layers
- `--border`, `--border2` — border colours
- `--text`, `--text2`, `--text3`, `--text4` — text hierarchy

---

## Feature Map

### Admin portal (role: `admin`)

| Sidebar item | Section component | Key capabilities |
|---|---|---|
| Dashboard | `Dashboard` | Stats, recent activity, quick links |
| Members | `MembersSection` | Add/edit/suspend members, view dues |
| Applications | `ApplicationsSection` | Review new membership applications, approve/reject |
| Membership Types | `MembershipTypesSection` | Manage types, pricing, availability |
| Committee | `CommitteeSection` | Display committee roles from member list |
| Vehicles | `VehiclesSection` | Register vehicles, pending approvals, REGO scanner |
| Meetings | `MeetingsSection` | Schedule meetings, agenda editor, minutes, RSVPs |
| Events & Shows | `EventsSection` | Events with RSVPs, check-in, trip logs, catering |
| Tasks | `TasksSection` | Task board (todo/in progress/done), assignments |
| Incident Reports | `IncidentsSection` | Log and manage safety/incident reports |
| Polls & Surveys | `PollsSection` | Create and run member polls |
| Commerce | `CommerceSection` | Shop, Parts Inventory, Parts Catalogue, Marketplace, Library |
| Website | `WebsiteSection` | Site content, news, newsletter builder, custom pages, gallery |
| Club Records | `RecordsSection` | Archive, Quality System, Library books |
| Recognition | `RecognitionSection` | Points & awards, competitions / show & shine |
| Administration | `AdministrationSection` | Finance, Contacts, Permissions, Categories, Bulk Import |
| **Account** | **`AccountSection`** | **Club Identity, Branding, Contact Details, Departmental Emails, Venues, Storage Locations** |
| My Profile | `ProfileSection` | Personal details, vehicle, password |

### Member portal (role: `member`)

Dashboard, My Membership, Committee, My Vehicles, Club Permits, Meetings, Events, Tasks, Incidents, Polls, Club Shop, Marketplace, Library, Club Archive, Points & Awards, My Profile.

### Public website

Home (hero, news preview, events preview), About, News, Events, Membership (with online application), Marketplace, Shop, Parts, custom pages.

---

## Storage Keys (SK object)

```js
members, cars, transfers, meetings, events, tasks, dues, rsvps, listings,
shopItems, orders, cart, news, siteContent, applications, books,
parts, partsInventory, qualityDocs, archiveDocs, emailDrafts, points,
forum, suppliers, pages, gallery, expenses, finSettings, permits, tripLogs,
categories, messages, checkins, contacts, competitions, pendingCars,
incidents, polls, partsCatalogue, partsEOI, invoices, accountSettings
```

All keyed as `"demo:" + keyname` in the demo instance.

---

## Recent Changes (newest first)

### 2026-04-13 — Storage location field in Parts Inventory
- Parts Inventory → Add Part modal now has a **Storage Location** field
- When storage locations exist in Account → Storage Locations, shows a dropdown; falls back to free-text input
- Location displays on each part card in inventory view (📦 icon)
- `PartsInventorySection` now accepts `accountSettings` prop (passed at both call sites)

### 2026-04-13 — Account section (6 tabs)
New **Account** nav item in admin sidebar (under Administration). Component: `AccountSection`. Storage key: `demo:accountSettings`.

- **Club Identity:** Registered name, trading name, ABN, ACN, incorporation number, state of incorporation, founded year, financial year end
- **Branding:** Logo upload (base64, with preview), primary / secondary / accent colour pickers; on save, updates `--burg`, `--gold`, `--burg-dk` CSS variables live
- **Contact Details:** Postal address, physical address, phone, general email, website, social media (Facebook, Instagram, YouTube, X/Twitter)
- **Departmental Emails:** Membership, events, technical, merchandise, newsletter — all optional, show general email as placeholder fallback
- **Venues:** Add/Edit/Delete club venues (name, address, suburb, state, postcode, notes, Google Maps URL). Venues populate a dropdown in the Meetings and Events location fields when any exist; falls back to free-text if none configured
- **Storage Locations:** Add/Edit/Delete parts storage locations (name, address, contact person, phone, notes). Used in Parts Inventory location field
- On save: updates live `CLUB_CONFIG` values (name, ABN, state, phone, emails, website) without page reload

### 2026-04-13 — Blank screen fix (missing `</tr>`)
- Fixed fatal Babel parse error in Block 2: a `<tfoot>` in the membership application invoice confirmation was missing `</tr>` before `</tfoot>`
- Added `window.onerror` red overlay to surface Babel/runtime errors visually (stays in the file as a permanent debugging aid)

### Earlier recent work
- Invoice generation on membership application submission; Invoices tab in Finance section; Payment Methods settings
- Parts Catalogue tab added to Commerce section
- Membership application system (multi-step public form, admin review, approve/reject flow)
- Hero image upload in Website admin; configurable hero name and tagline
- Membership types loaded dynamically from storage (not hardcoded)

---

## Testing / Validation

Since there is no test suite, validate JSX parses after any significant edit:

```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('index.html','utf8');
const babel = require('/tmp/node_modules/@babel/standalone/babel.js');
const re = /<script type=\"text\/babel\"[^>]*>([\s\S]*?)<\/script>/g;
let m, blocks = [];
while((m = re.exec(html)) !== null) blocks.push(m[1]);
blocks.forEach((b,i) => {
  try { babel.transform(b,{presets:['react']}); console.log('Block',(i+1),': OK'); }
  catch(e) { console.log('Block',(i+1),': ERROR:', e.message.split('\n')[0]); }
});
"
```

If `@babel/standalone` isn't available locally: `cd /tmp && npm install @babel/standalone`

**Common JSX pitfalls:**
- Every `<tr>` inside `<tbody>`/`<tfoot>`/`<thead>` must be explicitly closed with `</tr>`
- Self-closing tags: `<input/>`, `<br/>`, `<img/>` — never `<input>` bare
- `class` → `className`, `for` → `htmlFor`
- Inline styles are objects: `style={{color:"red"}}` not `style="color:red"`

---

## Deploy

```bash
git add index.html
git commit -m "Description of change"
git push origin demo
```

Netlify picks up the push automatically. No build step. Deployment takes ~30 seconds.

---

## Demo Credentials

| Role | Email | Password |
|---|---|---|
| Admin | admin@demo.club | admin123 |
| Member | sam@demo.club | member123 |

---

## Planned / In Progress

The following have been discussed or are logical next steps based on current work:

### Short-term
- **Edit part** — Parts Inventory cards currently only have Delete; an Edit button to update name, price, qty, location etc. is needed
- **Account settings propagation** — the logo saved in Account → Branding should replace the hardcoded `RccaLogo` SVG in the sidebar and dashboard header
- **Venue "View on Map" in meeting/event detail pages** — currently the Maps URL is stored but not surfaced in the read-only meeting/event view, only in the edit form
- **Storage location on parts cards (member view)** — location currently shows on admin inventory cards; consider whether members should see it too

### Medium-term
- **Financial year end from Account settings** — the `financialYearEnd` field stored in `accountSettings` should drive the Finance section's year-end calculations (currently hardcoded to 30 June)
- **Club name/branding live update** — sidebar header and browser tab title currently read `CLUB_CONFIG.name` at render time; after an Account save the name updates in memory but the sidebar doesn't re-render unless the tab changes
- **Newsletter email integration** — the Newsletter Builder currently creates HTML content but has no send/delivery mechanism
- **Bulk export** — no CSV/Excel export exists for members, dues, or financial data
- **Permit expiry notifications** — Club Permits section tracks expiry dates but there's no automated reminder workflow

### Architectural notes for future work
- Adding a new data entity requires: (1) add to `SK`, (2) add `useState`, (3) add to `Promise.all` destructuring + array, (4) set state in the effect body, (5) add to `sp` object
- The file is already large (~16k lines). Consider whether any new major features belong in a separate script block (Block 3 is the natural extension point)
- Babel Standalone processes blocks independently — function names declared in Block 1 are available in Blocks 2 and 3 via the global scope (no imports needed)
- `uid("PREFIX")` generates IDs; `today` is a `const` string `YYYY-MM-DD`; `yr` is the current year number
