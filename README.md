# All Hazard Plan — 5 Manhattan West

Mobile-first emergency incident response app for Fire & Life Safety Directors,
Engineers, Property Managers, and Security at 5 Manhattan West (Brookfield Properties).

Single self-contained file: **`index.html`**. No build step, no backend, no dependencies
(Google Fonts load from CDN; the app still works offline with system-font fallback).

Current version: **v1.3.2**

---

## Deploy (GitHub → Netlify)

1. Put `index.html` at the repo root. Keep the repo **private** (see Security below).
2. Netlify → **Add new site → Import from Git** → select this repo.
   - **Build command:** *(leave empty)*
   - **Publish directory:** `/` (root)
3. Every push to `main` auto-deploys in ~30s.
   - Optional: use a `staging` branch for a preview URL before promoting to production.

That's it — Netlify just serves the static file.

---

## How content is stored (read this before entering content)

There are **two** places content can live, and they are not the same:

| | Baked into `index.html` (`DEFAULT_DATA`) | Entered via the in-app Admin panel |
|---|---|---|
| Where it lives | The source file, committed to this repo | The browser's `localStorage` (`ahp-data`) |
| Who sees it | **Every device** that opens the site | **Only that one browser on that one device** |
| Survives cache clear / new device | Yes | No — falls back to the baked defaults |
| Shared to staff automatically | Yes | No |

**Implication:** content typed into the Admin panel is *device-local*. To make content
permanent and visible to everyone, it must be **baked into `DEFAULT_DATA`** and committed.

**Recommended workflow**
- **Canonical content** (incident types, BIC, tenants) lives in `DEFAULT_DATA` in the repo.
- The **Admin panel** is the live tweak/preview layer on top.
- Use **Admin → Export JSON** to back up any in-app edits; commit that file (e.g. `seed.json`)
  so nothing is trapped in a browser.

Already baked in: Elevator Entrapment, the FDNY Building Information Card, the tenant
directory, and admin credentials.

---

## Admin access

- Reached via the profile icon (top-right).
- Credentials are **hardcoded** in `index.html`.
- The Admin panel edits all content and exports/imports JSON backups.

---

## Security notes

- **Keep the repo private.** The file contains the admin password, staff emergency
  numbers, tenant contacts, and the FDNY BIC.
- **The deployed page is public-source.** GitHub privacy protects the repo, not the
  served page. Anyone with the Netlify URL can View Source / open DevTools and read the
  hardcoded password. Treat it as "keeps honest people out," not real security.
- If this ever becomes tenant- or guard-facing, move auth server-side (or at minimum a
  hashed check) instead of a plaintext client-side password.

---

## Data model (quick reference)

`localStorage['ahp-data']`:
```
appData = {
  buildingName,
  incidents: { [id]: { id, name, icon, tagline, definition,
                       announcements:[{id,title,script}],
                       roles:[{id,name,color,actions:[{id,stepNum,text,how,why,critical}]}],
                       contacts:[{label,number,emergency}] } },
  sections: {
    intro, eapScripts, buildingInfo,
    bic:     { bin, address, aka, revised, groups:[{id,title,rows:[{label,value}]}] },
    tenants: [ { id, name, type:'Office'|'Retail', floors, store,
                 contacts:[{name,title,office,emergency,email}] } ],
    floorPlans, resources
  }
}
```
Checklist tick state is stored separately in `localStorage['ahp-checklist']`.

---

## Changelog

- **v1.3.2** — Fixed the genie suppressing valid floor-based answers: the refuse gate counted
  only word matches, so "what stairways lead to street level" was rejected even though the right
  stairs (serving 1st / discharging at W 31–33 St) had ranked. A floor match now counts as
  evidence; "level" is treated as filler; and floor-1 / street-exit rows carry street/ground
  vocabulary so "street" matches directly. Guarded by a new phrasing test.

- **v1.3.1** — Genie natural-language tuning so common phrasings resolve: expanded the
  stopword list so filler words ("how many", "used for", "lead to") stop diluting scores;
  added concatenated name matching ("wholefoods" -> Whole Foods Market, "jpmorgan" -> JP
  Morgan Chase); mapped street/ground/lobby to floor 1 so "what elevators lead to the street"
  returns ground-serving banks; added a floor count to tenant answers ("Amazon - ... - 4
  floors"); made "where is X" favor the tenant's location over individual people; and added an
  exact-name boost so a bare tenant name outranks partial-name tenants (Amazon over
  Weworks-Amazon). Freight/service treated as synonyms. Guarded by 5 new phrasing tests.

- **v1.3.0** — Added **Ask the Plan** genie: an offline, deterministic question-answering
  layer over all plan content (incidents, roles, BIC, tenants, contacts). It returns stored
  values **verbatim**, shows a **source citation** for every answer, and attaches a **reliability
  score (0-100) + tier** (High/Medium/Low). It **detects compound/comparison questions** and
  answers each part separately, surfaces **how-to-ask hints** and starter chips, and **refuses
  rather than guesses** when confidence is low. No external AI, no network calls, no prompt to
  inject — answers can only come from this plan's data. A documented, inert-by-default
  `REMOTE_DATA_URL` sync hook is included so content can auto-refresh when online while staying
  fully usable offline (last-known-good).

- **v1.2.1** — Hardcoded admin credentials; removed plaintext hint from login screen.
- **v1.2.0** — Added FDNY Building Information Card (structured, readable) and Tenant
  Directory (Office/Retail, tap-to-call); non-destructive migration for existing installs.
- **v1.1.0** — Contacts editor (admin), incident search/filter, reset-checklist control,
  password-validation fix.
- **v1.0** — Initial MVP: incidents, role checklists, announcements, JSON export/import.
