# Summer Opportunities Directory — Technical Documentation

**File:** `summer-opportunities.html`
**Type:** Single self-contained static HTML file (HTML + CSS + JavaScript, no build step, no external JS frameworks, no server/backend)
**Maintained by:** The Stony Brook School Academic Council

This document describes the site as currently built, in enough detail that it could be reconstructed from scratch. It also contains a **pending, unapproved spec** for the next feature (program comparison) at the end — see the note there before building it.

> **Process rule (per site owner):** Any future change to the website must first be reflected in this documentation, presented for confirmation, and only implemented after approval. Update this file *before* editing `summer-opportunities.html`, not after.

---

## 1. Overview

A single-page directory of summer programs for high school students, filterable by grade, subject, format, cost, location, and selectivity. Built as one portable `.html` file — no installation, no dependencies beyond two Google Fonts loaded over CDN, works by double-clicking the file or hosting it anywhere static files are served.

### 1.1 Design goals
- Zero build tooling — must open directly in a browser.
- One file — logo and all data embedded inline (no separate asset files to lose track of).
- Fast filtering with plain JavaScript (no reactive framework) over a small, in-memory array of program objects.
- Mobile-first responsive behavior with a slide-in filter drawer below 860px width.

### 1.2 Non-goals
- No persistence (no localStorage/database) — all state resets on page reload.
- No backend — the "database" is a hardcoded JS array in the file itself.
- No routing/multi-page navigation.

---

## 2. Tech stack

| Layer | Choice |
|---|---|
| Markup | Plain HTML5 |
| Styling | Plain CSS3 with custom properties (`:root` variables), no preprocessor, no framework |
| Behavior | Vanilla JavaScript (ES6+), no libraries |
| Fonts | Google Fonts CDN: **Source Serif 4** (headings/serif accents) and **IBM Plex Sans** (UI/body) |
| Images | Logo embedded as a base64 `data:image/png;base64,...` URI directly in the `<img src>` — no external image file |

No npm, no bundler, no React. This is intentional so the file can be handed to anyone and opened with zero setup.

---

## 3. File structure (single file, four sections in order)

```
<!DOCTYPE html>
<html>
<head>
  <meta + title + Google Fonts link>
  <style> ... all CSS ... </style>
</head>
<body>
  <header class="top"> ... hero: logo, title, tagline, grade chips, search ... </header>
  <div class="filter-overlay-bg">                <!-- mobile drawer backdrop -->
  <div class="layout">
    <aside class="filters" id="filterPanel"> ... sidebar filters ... </aside>
    <main>
      <div class="results-head"> ... count + sort ... </div>
      <div class="card-list" id="cardList"> ... JS-rendered program cards ... </div>
      <div class="empty-state" id="emptyState"> ... </div>
    </main>
  </div>
  <footer class="site-footer"> ... disclaimers + contact ... </footer>
  <script> ... PROGRAMS data + all app logic ... </script>
</body>
</html>
```

---

## 4. Design system

### 4.1 Color palette (CSS custom properties, defined in `:root`)

| Variable | Hex | Used for |
|---|---|---|
| `--paper` | `#F6F5F0` | Page background |
| `--paper-raised` | `#FFFFFF` | Cards, header, sidebar, inputs |
| `--ink` | `#1C231D` | Primary text |
| `--ink-soft` | `#5B655C` | Secondary/meta text |
| `--hairline` | `#DDD9CC` | Default borders/dividers |
| `--hairline-strong` | `#C7C2B0` | Interactive element borders (inputs, chips) |
| `--green` | `#2F4A3D` | Primary accent — active states, links, grade badge |
| `--green-dark` | `#233A2F` | Headings (h1, program names) |
| `--ochre` | `#B9812C` | Selectivity dot ("Selective"), note-box border/text base |
| `--ochre-soft` | `#F1E3C7` | Note-box background |
| `--brick` | `#A8402C` | Urgency/flag color — selectivity dot ("Very selective"), flag badges |
| `--brick-soft` | `#F3DCD4` | Flag-box background |
| `--slate` | `#5B6660` | (reserved, currently mirrors ink-soft) |
| `--focus` | `#2F4A3D` | `:focus-visible` outline color |

Selectivity dot color mapping (in JS, `SELECTIVITY_DOT_CLASS`):
- Very selective → brick
- Selective → ochre
- Somewhat selective → `#7C9A6F` (sage green, inline not a variable)
- Less selective → `#8FA88A` (lighter sage, inline)
- Unknown → `#B7B2A2` (neutral grey-tan, inline)

### 4.2 Typography
- **Headings / program names / brand title:** `'Source Serif 4', serif` — set via a `.serif` utility class and directly on `h1`, `h2`, `h3`.
- **Everything else (body, labels, buttons, inputs):** `'IBM Plex Sans', sans-serif` — set as the `body` default.
- No uppercase/letter-spaced "eyebrow" labels — all copy is sentence case by design choice (avoids a templated/corporate feel).

### 4.3 Component style conventions
- **Border radius:** small throughout — `3px`–`4px` on cards/buttons/inputs, `20px` (pill) only on the toggle switch track and small badges.
- **Borders over shadows:** components are distinguished by a 1px `--hairline` border, not box-shadow. This is a deliberate "index card / ledger" aesthetic rather than a soft SaaS-card look.
- **Badges:** small pill-shaped `<span class="badge">` elements, outlined (not filled), used for grade range, "Free", "Pre-college", "U.S. only" flags.
- **Motion:** minimal — chevron rotation on card expand (`transform: rotate(180deg)`, 0.18s), card body `max-height` transition (0.22s) for the accordion effect, and a `translateX` slide for the mobile filter drawer (0.22s). No decorative animation elsewhere.

---

## 5. Page sections in detail

### 5.1 Header / hero (`<header class="top">`)
Contains, top to bottom:
1. **Brand row** (`.brand-row`): 64×64px logo image + a text block with:
   - `.eyebrow` — "The Stony Brook School Academic Council" (small, `--ink-soft`)
   - `<h1>` — "Summer Opportunities Directory"
2. **Tagline** (`<p class="tag">`) — one paragraph explaining the tool and crediting the Academic Council.
3. **Grade hero** (`.grade-hero`) — the primary interactive element, placed prominently because "what grade am I in" is the single highest-value filter:
   - Label: "I'm currently in grade:"
   - Five pill buttons (`.grade-chip`): All grades, 9th, 10th, 11th, 12th. Exactly one is `.active` at a time (starts on "All grades").
4. **Search row** (`.search-row`) — single text input, placeholder "Search by program name, keyword, or field…".

### 5.2 Filter sidebar (`<aside class="filters" id="filterPanel">`)
Desktop: `position: sticky; top: 20px` inside the two-column `.layout` grid (`grid-template-columns: 270px 1fr`).
Mobile (≤860px): becomes a fixed-position off-canvas drawer sliding in from the right (`transform: translateX(100%)` → `translateX(0)` when `.open` is added), behind a semi-transparent backdrop (`.filter-overlay-bg`).

Contains, in order, each as a `.filter-group` with a heading:
1. **Subject** — checkboxes for each of `CATEGORY_ORDER`.
2. **Format** — checkboxes for `FORMAT_ORDER` (In-person / Online / Hybrid).
3. **Approx. cost** — checkboxes for `COST_ORDER` tiers (Free / Under $2,000 / $2,000–$6,000 / $6,000+ / Varies).
4. **Location** — checkboxes for `REGION_ORDER`.
5. **Selectivity** — checkboxes for `SELECTIVITY_ORDER`.
6. **Two toggle switches** (`.switch`, styled checkbox): "Hide pre-college campus programs" and "Hide U.S.-only (domestic) programs".
7. **Reset all filters** button.

All checkbox/toggle groups are **multi-select** (checking multiple boxes in one group is OR logic within that group; different groups combine with AND logic — see §7.2).

### 5.3 Results area (`<main>`)
- **Results head:** left side shows `<strong id="countNum">` + "program(s)"; right side (desktop only, hidden on mobile in favor of the mobile filter bar's own sort dropdown) is a `<select id="sortSelect">`.
- **Mobile filter bar** (`.mobile-filter-bar`, shown only ≤860px): a "Filters" button with a live count badge (`#activeFilterCount`) that opens the drawer, plus a duplicate sort `<select id="sortSelectMobile">` kept in sync with the desktop one.
- **Card list** (`#cardList`): vertically stacked list (not a grid) of program cards — chosen over a grid because rows of metadata read better scanned top-to-bottom than in variable-height grid cells.
- **Empty state** (`#emptyState`): shown when zero programs match; includes a "Reset all filters" button.

### 5.4 Program card (rendered by `renderCard(p)`)
Collapsed state (`.card-head`, clickable):
- Program name (`.card-name`, serif)
- Meta row (`.card-meta`): grade badge, selectivity dot + label, format, short location, cost (or "Free" badge), optional "Pre-college" flag badge, optional "U.S. only" flag badge
- Chevron icon (rotates 180° when open)

Expanded state (`.card-body`, animated open via JS-computed `max-height`):
- Full description paragraph
- A definition-list grid (`.detail-grid`) with fields: Location, Cost, Deadline, Subject area(s), Requirements
- **Official website link** (`.program-link`) — a small "Visit official site ↗" link placed directly under the description, `target="_blank" rel="noopener"`. Rendered only when `p.url` is not `null`; omitted entirely (no placeholder text) when a link isn't available, so the absence doesn't read as a broken feature.
- A `.note-box` (ochre) with the program's `gradeNote` — explains any ambiguity in how eligibility was interpreted
- An optional `.flag-box` (brick) if the program is pre-college and/or domestic-only, with explanatory text

### 5.5 Footer (`<footer class="site-footer">`)
Three disclaimer paragraphs (informational-only notice, pre-college programs caution with an external link, and a note on how gaps in the data were filled via research) plus a contact paragraph: "Any questions?" pointing to the Academic Council email and to `college.counseling@sbs.org` for corrections/additions.

---

## 6. Data model

All content lives in one JavaScript array, `PROGRAMS`, defined near the top of the `<script>` block. There is no external data file — **this array is the entire database.**

### 6.1 Program object schema

```js
{
  id: "kebab-case-unique-id",           // string, unique, used as DOM key & open/closed tracking
  name: "Full Program Name",             // string, displayed as card title
  url: "https://official-program-page.example.edu", // string or null — official program page; null if unverified
  categories: ["STEM", "Business"],      // array of strings, each MUST be one of CATEGORY_ORDER
  grades: [10, 11],                      // array of integers from {9,10,11,12} — CURRENT grade(s) eligible
  gradeNote: "Explanation of eligibility, especially if grades[] required interpretation or is unconfirmed.",
  format: "In-person",                   // exactly one of FORMAT_ORDER: "In-person" | "Online" | "Hybrid"
  region: "Northeast",                   // exactly one of REGION_ORDER
  location: "Full location string, City, State", // free text, full detail; card preview shows first 2 comma-parts only
  costTier: "high",                      // exactly one of: "free" | "low" | "mid" | "high" | "varies"
  costDisplay: "$8,000 (financial aid available)", // free text, human-readable, shown in expanded detail
  selectivity: "Selective",              // exactly one of SELECTIVITY_ORDER
  deadlineText: "March 1, 2026",         // free text, shown in expanded detail
  deadlineSort: 3,                       // integer 1–12 = month; 13 = "rolling"; 14 = "not stated/TBD" — used ONLY for sorting
  preCollege: true,                      // boolean — drives the "Pre-college" badge + hide toggle + flag-box copy
  domesticOnly: false,                   // boolean — drives the "U.S. only" badge + hide toggle + flag-box copy
  description: "1–2 sentence plain-language summary of what the program is.",
  requirements: "What the application requires (essays, recs, tests, fees, etc.)."
}
```

**On `url`:** every one of the 49 programs now has a verified official link — 14 were confirmed via research, and the remaining 35 were supplied directly by the site owner. Cards and the compare table both render the link only when present; the schema still supports `null` for any future program added without a confirmed link, so a wrong link is never guessed at. One exception is handled in free text rather than the `url` field: **American Legion Boys State / Girls State** covers two separate organizations with two separate links; `url` holds the Boys State link, and the Girls State link (which varies by state — e.g. empiregirlsstate.org for New York) is called out in that program's `requirements` field instead, since the schema only supports one link per program.

### 6.2 Controlled vocabularies (also defined in the script, used both for filter-building and validation-by-convention)

```js
CATEGORY_ORDER   = ["STEM","Language & Linguistics","Arts","Business","Leadership",
                     "Underrepresented Students","Liberal Arts & Humanities","Other"]
FORMAT_ORDER     = ["In-person","Online","Hybrid"]
COST_ORDER       = [{key:"free",label:"Free"}, {key:"low",label:"Under $2,000"},
                     {key:"mid",label:"$2,000–$6,000"}, {key:"high",label:"$6,000+"},
                     {key:"varies",label:"Varies / not published"}]
REGION_ORDER     = ["Northeast","Midwest","South","West","Multiple","Online","International"]
SELECTIVITY_ORDER= ["Very selective","Selective","Somewhat selective","Less selective","Unknown"]
```

**To add a new program:** append a new object to `PROGRAMS` following the schema above, using only values from the controlled vocabularies for `categories`, `format`, `costTier`, `region`, and `selectivity`. No other code changes are required — filters, counts, and rendering all derive from this array automatically.

**Current dataset size:** 49 programs (as of this version).

---

## 7. Application logic

### 7.1 State object

A single mutable `state` object holds all UI state (no framework state management):

```js
state = {
  grade: "all",                 // "all" | "9" | "10" | "11" | "12" (string, matches chip data-attribute)
  categories: new Set(),        // selected subject checkboxes
  formats: new Set(),
  costs: new Set(),
  regions: new Set(),
  selectivities: new Set(),
  hidePreCollege: false,
  hideDomestic: false,
  search: "",                   // lowercase, trimmed
  sort: "deadline",              // "deadline" | "name" | "cost-asc" | "selectivity"
  openCards: new Set()           // ids of currently-expanded cards, so they survive re-render
}
```

Every filter/search/toggle change calls `render()`, which recomputes the filtered+sorted list and redraws the card list. `openCards` is what makes expand/collapse survive a full re-render (e.g. after typing in the search box while a card is open).

### 7.2 Filtering logic (`matches(p)`)

A program `p` is included if **all** of the following pass (AND across categories; OR within each multi-select group via `.some()`/`.has()`):

1. If a specific grade is selected, `p.grades` must include it.
2. If any subject checkboxes are checked, at least one of `p.categories` must be checked.
3. If any format checkboxes are checked, `p.format` must be one of them.
4. If any cost checkboxes are checked, `p.costTier` must be one of them.
5. If any region checkboxes are checked, `p.region` must be one of them.
6. If any selectivity checkboxes are checked, `p.selectivity` must be one of them.
7. If "hide pre-college" is on, `p.preCollege` must be false.
8. If "hide domestic-only" is on, `p.domesticOnly` must be false.
9. If there's search text, it must appear (case-insensitive substring) in the concatenation of `name + categories + description + location`.

### 7.3 Sorting logic (`sortPrograms(list)`)
- `"deadline"` (default): ascending by `deadlineSort`.
- `"name"`: alphabetical by `name` (`localeCompare`).
- `"cost-asc"`: by `costTier` in the fixed order free < low < mid < high < varies.
- `"selectivity"`: by fixed order Very selective < Selective < Somewhat selective < Less selective < Unknown.

### 7.4 Rendering
- `render()` — top-level: recompute filtered set, update the count text, toggle empty-state visibility, update the mobile active-filter-count badge, then call `renderCardsOnly()`.
- `renderCardsOnly()` — clears and rebuilds `#cardList` from the current filtered+sorted list, calling `renderCard(p)` per program, then (on next animation frame) sets `max-height` inline on any `.card.open .card-body` so the CSS transition has a concrete pixel value to animate to/from.
- `renderCard(p)` — builds the card DOM as an HTML string via template literal, attaches a click listener on `.card-head` that toggles the card's id in/out of `state.openCards` and re-renders just the card list (not the whole page).

### 7.5 Filter UI construction
`buildCheckGroup(containerId, items, stateSet, labelFn, keyFn)` is a small generic helper called once per filter group (subject, format, cost, region, selectivity) at page load. It generates the checkbox rows from the controlled-vocabulary arrays and wires each one's `change` event to add/remove from the relevant `state` Set and call `render()`. This means **adding a new subject category, format, region, etc. only requires adding it to the relevant `_ORDER` array** — the checkbox UI is generated, not hand-written per option.

### 7.6 Mobile drawer
`openDrawer()` / `closeDrawer()` toggle the `.open` class on `#filterPanel` and `.show` on `#overlayBg`. Triggered by the mobile "Filters" button, the drawer's own close (×) button, and clicking the backdrop.

### 7.7 Reset
`resetAll()` clears every field of `state` back to defaults, unchecks every checkbox/toggle in the DOM, resets the grade chip to "All grades", clears the search input, and re-renders. Wired to both the sidebar's "Reset all filters" button and the empty-state's "Reset all filters" button.

---

## 8. Responsive behavior

Single breakpoint at **860px** (`@media (max-width: 860px)`):
- `.layout` collapses from a 2-column grid (`270px 1fr`) to a single column.
- `.filters` sidebar becomes the fixed off-canvas drawer described in §5.2/§7.6.
- `.mobile-filter-bar` (Filters button + sort dropdown) becomes visible; the desktop sort dropdown in `.results-head` is effectively redundant on mobile screens (both exist in the DOM and are kept in sync, but only one is visually reachable at a time depending on width).
- `.search-row` expands to full width.

No other breakpoints are used — the layout is designed to work acceptably from ~360px mobile up through desktop via this single collapse point plus fluid `clamp()` sizing on the `h1`.

---

## 9. Branding / logo embedding

The Stony Brook School seal is embedded directly as a base64-encoded PNG data URI in the `<img src="data:image/png;base64,...">` attribute of `.brand-row img`. The source image was resized to 240×240px before encoding (from an original 1600×1600px upload) to keep the file size reasonable — at 240×240 the encoded string is ~101KB of the file's ~167KB total.

**To change the logo:** re-encode a new image as base64 (`base64 -w0 logo.png`) and replace the string between `base64,` and the closing quote in the `<img>` tag. No other file is referenced.

---

## 10. Known content caveats (carried into any rebuild)

- The dataset combines the school's original PDF listing with independent web research to fill gaps in published grade eligibility, cost, and deadlines. Every program's `gradeNote` field flags where this happened.
- A handful of programs (e.g. UPenn ESAP, NYU HSLI, ieSoSC) had no publicly available grade-eligibility information at the time of writing; these are set to all grades `[9,10,11,12]` with a note to confirm directly, rather than guessed at.
- Costs (`costDisplay`) are shown as approximate/rounded and are separately bucketed into `costTier` for filtering purposes only — the tier boundaries are a simplification and should not be read as precise.

---

## 11. How to extend this site (cookbook)

| Task | What to change |
|---|---|
| Add a program | Append an object to `PROGRAMS` following §6.1's schema |
| Add/update a program's official link | Set `url` on that program's object; use `null` if no confidently-verified official page exists |
| Add a new subject category | Add the string to `CATEGORY_ORDER`; use it in any program's `categories[]` |
| Add a new region | Add to `REGION_ORDER`; use in `region` field |
| Change a color | Edit the relevant `--variable` in `:root` |
| Change fonts | Swap the Google Fonts `<link>` and the `font-family` values in `body`/`.serif` |
| Change the mobile breakpoint | Edit the `860px` value in the one `@media` query and its two references |
| Change sort options | Add to the `SORT_OPTIONS` array (each needs `key` + `label`) and a matching branch in `sortPrograms()` |

---

## 12. Compare view

**Status: approved and implemented.** Final spec below (superseding the original proposal — pre-college row removed, entry point confirmed as search-only, no URL persistence).

### 12.1 Entry point
A **"Compare"** button in `.results-head`, next to the sort dropdown (desktop), and a matching button in the mobile filter bar. Clicking it opens a full-screen overlay on top of the current page (same file, no navigation/page change). Selection is **purely search-driven** — there is no "add to compare" affordance on the main program cards; the only way to populate a slot is to search for it inside the overlay.

### 12.2 The compare overlay
- A modal (`#compareOverlay`) with a dimmed backdrop, closable via an × button, clicking the backdrop, or the Escape key.
- Inside: two independent search boxes side by side (stacked on mobile), labeled "Program A" and "Program B".
- Typing in either box live-filters a dropdown of matching program names (substring match against `name`, same style as the main search). Selecting a result fills that slot.
- Each filled slot shows a small "×" to clear it and search again.
- **No persistence:** closing the overlay or reloading the page clears both slots, consistent with the rest of the site having no saved state.

### 12.3 The comparison layout — how "characteristics line up"
Rather than two fully independent free-floating cards (where paragraph lengths could cause fields to drift out of alignment), the comparison is built as a **shared two-column table**: one label column, then a value column for Program A and a value column for Program B, so every attribute sits on its own row and is guaranteed to line up regardless of text length. Rows, in order:

1. Program name (header row, not a labeled row)
2. Grade eligibility
3. Subject area(s)
4. Format
5. Location
6. Cost
7. Selectivity
8. Deadline
9. U.S. citizens/permanent residents only? (Yes/No)
10. Official website (rendered as a link when `p.url` is present, otherwise "Not available")
11. Description
12. Requirements

("Pre-college?" removed per confirmation — not shown as a comparison row.)

If only one slot is filled, that column shows its data and the other column shows a placeholder ("Search and select a program to compare"). If neither slot is filled, both columns show the placeholder and the table rows are hidden until at least one program is selected.

### 12.4 Data & state
```js
compareState = { a: null, b: null }   // program id or null, per slot
```
New functions: `openCompare()`, `closeCompare()`, `renderCompareSuggestions(slot, query)`, `selectCompareProgram(slot, id)`, `clearCompareSlot(slot)`, `renderCompareTable()`. All read from the existing `PROGRAMS` array — no new data source needed.

### 12.5 New CSS
New rules added for: `.compare-overlay` (fixed, full-screen, dark backdrop), `.compare-modal` (centered white panel, max-width, scrollable), `.compare-columns` (two-column search row, stacks on mobile), `.compare-search-box` (input + suggestion dropdown), `.compare-suggestions` (absolute-positioned list under each input), `.compare-table` (CSS grid: label column + 2 value columns), `.compare-placeholder` (muted italic text shown for an empty slot).

---

## 13. Council login & on-site editing (Firebase) — implemented

**Status: built.** Backend = Firebase project `sbs-academic-council`. Site is hosted at `https://sbsacademiccouncil.github.io/Summer-Opportunities/`. Firebase SDK pinned to v10.9.0, loaded as ES modules from `gstatic.com` (the one exception to the "self-contained file" rule — this and the Google Fonts link are the only two external network dependencies the page has).

### 13.0 One-time setup still required outside the code (site owner only)
I cannot reach your live Firebase project from here, so these three things must be done by hand in the Firebase console before login/editing will actually work when the page loads:
1. **Authentication → Settings → Authorized domains** — add `sbsacademiccouncil.github.io`.
2. **Firestore Database → Rules** — paste in the rules from §13.5 below (replacing the defaults).
3. **Firestore Database → Data → Start collection `admins`** — add one document per Council editor, where **the document ID is the exact email** (no fields required, an empty document is fine): `terrence.wang@sbs.org`, `sarah.fay@sbs.org`, `meghan.fay@sbs.org`.
4. When uploading to the `Summer-Opportunities` GitHub repo, name the file `index.html` at the repo root (or in whatever path GitHub Pages is configured to serve) so it loads at the URL above.

The `programs` collection does **not** need manual seeding — see §13.9.

### 13.1 Why this is a bigger change than anything so far
Every prior feature (compare view, links) worked by adding more JavaScript on top of the same static file. This one can't, because the entire point is that **the browser can no longer be trusted to enforce "only Council members can edit."** A static file has no server standing behind it to check credentials, so this necessarily turns the site from *one self-contained file* into *one file that talks to a live backend (Firebase)*. Two consequences, stated plainly:
- **The site no longer works fully offline.** It needs to reach Firebase to load program data and to check who's signed in.
- **The site no longer works with zero setup.** It depends on a real Firebase project existing, configured with real values only the project owner can generate — hosting it also requires a real `https://` address (Google Sign-In does not work from a file opened directly off a computer, and Google Sites' sandboxed custom-HTML embed generally breaks OAuth redirects, so a plain static host — Firebase Hosting, GitHub Pages, or the school's own web server — is needed instead).

### 13.2 New data model: Firestore replaces the embedded `PROGRAMS` array
- A Firestore collection named `programs`, one document per program, using **the same field schema as §6.1** (`name`, `url`, `categories`, `grades`, `gradeNote`, `format`, `region`, `location`, `costTier`, `costDisplay`, `selectivity`, `deadlineText`, `deadlineSort`, `preCollege`, `domesticOnly`, `description`, `requirements`) — the document ID takes over the role the current `id` field plays.
- The page subscribes to this collection live (Firestore's `onSnapshot`), so a Council member's edit appears for every visitor automatically, with no refresh needed.
- The existing 49 programs get migrated into Firestore once the project exists, via a one-time seed script — not re-entered by hand.

### 13.3 The Council allowlist — designed specifically so it needs no code changes
- A second Firestore collection, `admins`, where **the document ID is a Council member's exact school email** (e.g. a document literally named `jane.doe@sbs.org`). The document's contents don't matter — its *existence* is what grants access.
- This collection is **not editable from the website itself** — by design, it can only be viewed/edited directly inside the Firebase console's Firestore data browser. This is what makes the roster "a short list you can edit later without touching code": adding a new Council member, or removing a graduating one, is adding/deleting one row in a table inside Firebase's own dashboard.
- On sign-in, a security rule permits a signed-in user to check only *their own* email against this collection — never the full list. If their document exists, they get edit access for that session; if not, they see the public read-only site.

### 13.4 Sign-in & permission flow
1. A small, deliberately unobtrusive link in the header's top-right reads **"Council editor sign-in"** — not a generic "Sign in" — specifically so ordinary visitors don't take it as something meant for them. It's styled as muted text, not a prominent button.
2. Clicking it opens Firebase Authentication's Google sign-in popup. Any school Google account can complete this step — **completing it does not by itself grant edit access.**
3. Immediately after sign-in, the site checks `admins/{their email}`:
   - **Found →** Edit mode activates: Edit/Delete controls appear on every card, plus "Add new program." The header link is replaced with "Editing as {email} · Sign out." All writes route through Firestore Security Rules that independently re-check the same `admins` document server-side, so an unauthorized write is rejected by Firebase itself even if someone tampered with the page's JavaScript, not merely hidden by the UI.
   - **Not found →** a dismissible banner appears: **"Signed in as {email} — this account isn't on the Council editor list. You're viewing the site as a regular visitor."** No edit tools are shown; the person is automatically signed back out after acknowledging the banner (or immediately, on a short delay), so they aren't left in a confusing half-signed-in state.
   - If the Google sign-in popup itself fails (closed by the user, blocked by the browser, network error), a small inline message explains that and invites retrying — it does not throw a raw JavaScript error to the console only.
- The site stays fully publicly viewable without any sign-in at all; only *editing* requires Council sign-in, matching "no one else [can edit]," not "no one else can view."

### 13.5 Firestore Security Rules (final text, ready to paste in — independent of your project's specific config values)
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /programs/{programId} {
      allow read: if true;
      allow write: if request.auth != null &&
        exists(/databases/$(database)/documents/admins/$(request.auth.token.email));
    }

    match /admins/{email} {
      allow get: if request.auth != null && request.auth.token.email == email;
      allow list, write: if false; // only editable via the Firebase console itself
    }
  }
}
```

### 13.6 Setup checklist (completed by the site owner)
Firebase web API keys are **not secret** — safe to have pasted here and safe to leave visible in the page's source; Firebase's security model relies entirely on the rules above, not on hiding the key. What was needed, now done:

1. Firebase project created at console.firebase.google.com (`sbs-academic-council`).
2. **Authentication → Sign-in method → Google provider** enabled.
3. **Firestore Database** enabled (rules from §13.5 to be pasted in under Firestore → Rules — see §13.0's remaining steps).
4. A **Web App** registered inside the project, `firebaseConfig` provided and now embedded in `summer-opportunities.html`.
5. Hosting domain confirmed: `https://sbsacademiccouncil.github.io/Summer-Opportunities/`.
6. Three Council member emails provided as the initial allowlist (added to Firestore per §13.0, step 3): `terrence.wang@sbs.org`, `sarah.fay@sbs.org`, `meghan.fay@sbs.org`.

### 13.7 Ownership note
Whoever creates the Firebase project should ideally be a role/advisor account rather than an individual student's personal login, so access isn't lost when that student graduates.

### 13.8 Explicitly out of scope unless requested
- Password-based login (Google Sign-In only).
- Per-edit change history/audit log ("last edited by ___ on ___") — Firestore doesn't track this out of the box; a small addition if wanted.
- An offline/static export snapshot for archiving, since the live version now requires Firestore to be reachable.

### 13.9 Status: implemented
§13.6's checklist is complete — real `firebaseConfig`, hosting domain (`sbsacademiccouncil.github.io`), and three sample Council emails were provided. Implementation below is now built into `summer-opportunities.html`.

### 13.10 Sign-in button is deliberately understated and explicitly labeled
Rather than a generic, prominent "Sign in" button that any visitor might be tempted to click, the button reads **"Council sign-in"**, sits small and unobtrusive in the top-right corner of the header (not the hero area), and carries a `title` tooltip ("For Academic Council editors only") on hover. The intent is that an ordinary visitor scanning the page has no reason to notice or click it, while a Council member looking for it can find it.

### 13.11 Handling a signed-in account that isn't on the allowlist
If someone completes Google sign-in but their email has no matching document in `admins`:
1. They are **not** left in a signed-in state — the app immediately signs them back out of Firebase Auth.
2. A dismissible red banner appears at the top of the page for a few seconds: *"`{their email}` is not on the Council editor list. Signing you out."*
3. The page reverts to exactly the normal signed-out view — no partial/broken state, no edit controls ever rendered for them at any point.
4. The same banner/sign-out path also handles a sign-in that errors or is cancelled, with an appropriate message.

### 13.12 One-time data seeding
Because Firestore can only be written to from a real browser session (not from this development sandbox), migrating the current 49 hardcoded programs into Firestore happens via a one-time **"Import starter list"** button, visible only in Council edit mode. Clicking it (after a confirmation dialog) writes all 49 programs from an embedded seed dataset into the `programs` collection in one batch. This is meant to be run once, by any Council member, right after the database is empty — running it again would overwrite existing documents with the original seed data, which the confirmation dialog explicitly warns about.

### 13.13 Editing, adding, and deleting programs
- Each card, when signed in as an authorized editor, shows an **"Edit program"** button in its expanded detail view.
- Clicking it (or the results-area **"Add new program"** button) opens a shared modal form with fields for every schema property in §6.1 (checkboxes for grades and subjects, dropdowns for the controlled-vocabulary fields, text/textarea inputs for everything else).
- Saving calls Firestore's `setDoc` directly from the browser; Firestore's own security rules (§13.5) independently re-verify the editor's email is on the allowlist before accepting the write, regardless of what the page's JavaScript sends.
- A new program's document ID is auto-generated by slugifying its name (lowercased, hyphenated, de-duplicated against existing IDs).
- Deleting a program is available from the same edit form, behind a confirmation dialog.
- All visitors see changes live within moments, since the page subscribes to the `programs` collection rather than polling or requiring a refresh.
