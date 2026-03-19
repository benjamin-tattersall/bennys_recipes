# Benny's Recipes — AI Agent Reference Document

> **HOW TO USE THIS FILE**
> At the start of every session: fetch this file via its raw GitHub URL, read it fully, then load the uploaded HTML. This file is the source of truth for project state, architecture, and next steps. Update it at the end of every session before handing back to the user.
>
> **Raw URL pattern:** `https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPO/main/bennys-recipes-backlog.md`

---

## 1. PROJECT SNAPSHOT

| Property | Value |
|---|---|
| Project name | Benny's Recipes |
| Current HTML file | `bennys-recipes-v37.html` |
| Type | Single-file HTML PWA |
| Hosting | GitHub Pages |
| Data backend | GitHub Gist (3 files: `recipes.json`, `bin.json`, `settings.json`) |
| Auth | User enters Gist ID + personal access token via 🔑 modal; stored in localStorage |
| Breakpoint | Mobile ≤600px, Desktop >600px |
| Design language | Warm off-white (`--cream`), terracotta accent (`--accent`), Playfair Display headings, Instrument Sans body |

---

## 2. ARCHITECTURE

### File structure (single HTML)
- All CSS, HTML, and JS in one file — no build step, no dependencies except Google Fonts
- GitHub Gist stores `recipes.json`, `bin.json`, `settings.json`
- localStorage caches all three for offline use
- Pending sync queue auto-flushes on reconnect

### Key JS globals
```
recipes[]         — live recipe array
bin[]             — soft-deleted recipes
undoStack[]       — in-memory undo (max 20), each entry: {recipes, bin, description}
redoStack[]       — in-memory redo
gistId, ghToken   — credentials from localStorage
pendingSync[]     — offline queue
scaleFactor       — current view scale (1, 0.5, 2, 3)
currentId         — recipe ID open in view modal
editId            — recipe ID open in edit modal (null = new)
showDrafts        — toolbar drafts toggle
groupBy           — current group-by key or null
subToggledMap{}   — tracks which substitute is active per ingredient in view
```

### Save flow
Every user action that modifies data calls `saveToGist(description)` or `saveLocally(description)`.
Both push the pre-save state onto `undoStack` with a human-readable description label before writing.
Undo/redo use silent variants (`saveToGistSilent` / `saveLocallySilent`) that skip the undo push.

---

## 3. DATA MODELS

### Recipe object
```json
{
  "id": 1700000000000,
  "name": "Recipe Name",
  "status": "draft | published",
  "scalable": true,
  "created": 1700000000000,
  "updated": 1700000000000,
  "tags": ["Dinner", "Korean"],
  "hours": 1,
  "minutes": 30,
  "serves": 4,
  "source": "",
  "youtube": "https://youtube.com/watch?v=...",
  "notes": "",
  "ingredients": [
    {
      "title": "Section name",
      "items": [
        {
          "name": "Garlic",
          "amount": 2,
          "unit": "clove",
          "prep": "minced",
          "noscale": false,
          "scaleOverrides": {
            "half":   { "amount": 1, "unit": "clove" },
            "double": null,
            "triple": null
          },
          "substitutes": [
            {
              "name": "Shallot",
              "amount": 3,
              "unit": "whole",
              "note": "if no garlic",
              "noscale": false,
              "scaleOverrides": {
                "half": null,
                "double": { "amount": 5, "unit": "whole" },
                "triple": null
              }
            }
          ]
        }
      ]
    }
  ],
  "method": [
    {
      "title": "Section name",
      "steps": [
        {
          "text": "Step text",
          "noscale": false,
          "scaleOverrides": { "half": null, "double": null, "triple": null }
        }
      ]
    }
  ]
}
```

### Serves field
- Exact: `"serves": 4`
- Range: `"serves": { "min": 2, "max": 4 }`

### Settings object (`settings.json`)
```json
{
  "customTagGroups": {},
  "retiredUnits": [],
  "customUnitGroups": {}
}
```

### Bin object (`bin.json`)
Array of recipe objects with an added `"deletedAt": timestamp` field.

---

## 4. UI STRUCTURE

### Header (left to right)
Logo | Search (desktop) | [toolbar below] | ⓘ Info | 🔑 Auth | 🗑 Bin | ⚙ Settings | ➕ Add Recipe
- All icon buttons: 34×34px square
- Add Recipe: shows label on desktop, icon-only on mobile (≤520px)

### Toolbar
Search icon (mobile) | Draft toggle | Undo ↩ | Redo ↪ | Filter | Group

### Modals
| Modal | Max-width | Trigger |
|---|---|---|
| View recipe | 980px | Card click |
| Edit/Add recipe | 900px | Add / Edit button |
| Info | 620px | ⓘ |
| Auth | 420px | 🔑 |
| Bin | 600px | 🗑 |
| Settings | 600px | ⚙ |
| Filter | 520px | Filter button |
| Group By | 380px | Group button |

### Ingredient form layout
**Desktop (>600px):** Ghost card header row (Ingredient / Amount / Unit / Prep centred over columns) then single-row input cards. Grid ratios: `40fr 10fr 20fr 30fr auto auto auto`. Sub-panel and scale override panel expand inside the same card border with tinted background.

**Mobile (≤600px):** 3-row card per ingredient. Row 1: full-width name. Row 2: amount (1/3) + unit (2/3), both height 38px. Row 3: prep + ⇄ ⊘ buttons (38×38px, matching input height). Corner × delete top-right of card. Substitute entries follow the same 3-row pattern.

### Default tag groups
```
Meal:    Breakfast, Brunch, Lunch, Dinner, Dessert, Snack, Side
Cuisine: Australian, Chinese, French, Greek, Indian, Indonesian, Italian,
         Japanese, Korean, Lebanese, Malaysian, Mexican, Middle Eastern,
         Spanish, Thai, Vietnamese
Type:    Bake, Bowl, Bread, Burger, Cake, Curry, Noodles, Pasta, Pie,
         Pizza, Roast, Salad, Sandwich, Seafood, Soup, Stew, Stir-fry,
         Sushi, Tacos, Toast
```

### Unit system
- Metric volume: `ml`, `L` — convert only within this family
- Cooking volume: `tsp`, `tbsp`, `cup` — convert only within this family
- Mass: `g`, `kg`
- Custom units: user-defined, can be retired/restored in Settings

---

## 5. BACKLOG

### Status key
`[ ]` Not started | `[~]` In progress | `[x]` Done | `[-]` Shelved

---

### CONFIRMED — work through in order

#### [ ] 1. Authentication overhaul
**Goal:** Replace manual token entry with a flow accessible to non-technical users.
**Options discussed:** OAuth "Sign in with GitHub", or URL-based encrypted token sharing.
**Blocked by:** Decision on distribution strategy.
**Notes:** Current localStorage token system works fine for personal use. Revisit when sharing with others becomes the priority.

#### [ ] 2. Individual recipe sharing
**Goal:** Share a single recipe in a clean readable format.
**Options discussed:**
- Share button → generates read-only standalone HTML or shareable link
- Copy-to-clipboard formatted text as simple fallback
**Notes:** Should work without requiring the recipient to have auth set up.

#### [ ] 3. Full site distribution
**Goal:** Someone else can take the app and set it up for their own recipe collection.
**Options discussed:** Downloadable HTML with guided first-run setup wizard built in.
**Dependency:** May depend on auth overhaul (#1).

#### [ ] 4. Serves display variants
**Goal:** Support "Makes 24 cookies" style yield expressions beyond "X serves".
**Notes:** Low priority. Currently always shows "X serves" or "X–Y serves".

#### [ ] 5. Imperial ↔ metric conversion
**Goal:** Toggle in recipe view to switch all quantities between unit systems.
**Notes:** Shelved previously. Tricky for non-convertible ingredients (e.g. "1 whole onion").

---

### IDEAS NOTED — not yet confirmed, raise when relevant

- **Print view** — clean printer-friendly recipe layout
- **Recipe import** — parse a URL or pasted text into a recipe using the Anthropic API
- **Ratings / cook notes** — private star rating or "I made this" note per recipe
- **Cook mode** — step-by-step full-screen view for active cooking, large text, one step at a time
- **Shopping list** — export or copy merged ingredient list from one or more selected recipes
- **Duplicate recipe** — clone an existing recipe as a starting point

---

### KNOWN ISSUES — shelved, not blocking

- **Filter dimming edge cases** — some combined filter states don't dim correctly; user happy to leave
- **Override unit manual entry** — scale override unit dropdowns don't support free-text entry like ingredient unit dropdowns; shelved

---

## 6. SESSION LOG

Append a new entry at the end of every session. Format: `### vXX — brief description` plus bullets of what changed. Most recent at the bottom.

### v1–v21 — Core build
Recipe CRUD, sectioned ingredients/method, tag system, scaling with overrides, serves range, draft/publish, group-by, filter, settings, YouTube embeds, mobile search collapse, info/auth split.

### v22–v23 — Offline + info/auth split
Separated ⓘ info from 🔑 auth modal. Offline sync queue with auto-flush on reconnect.

### v24 — Bin
Soft delete to `bin.json`. Bin modal: restore / permanent delete / empty bin. Header icon turns red when non-empty.

### v25–v26 — Undo/Redo
In-memory stack (20 steps). Toolbar buttons + ⌘Z/⌘⇧Z. Descriptive toasts per labelled action.

### v27–v29 — Substitute ingredients
Multiple subs per ingredient, each with own scale overrides. View: 4th column ⇄ → popup chooser. Active sub: bold orange + asterisk. Sub note italic beneath.

### v30 — Display fixes round 1
Wider desktop modals. Square header buttons. Add Recipe label hides on mobile.

### v31–v36 — Ingredient form redesign
Desktop: single-row grid (40/10/20/30 ratios) with ghost card column headers. Mobile: 3-row card layout, corner delete, equal-height inputs, sub entries matching same pattern. Embedded credentials attempted and reverted (GitHub secret scanning).

### v37 — Auth revert + info guide
Removed embedded credentials. Reverted to localStorage auth. Rewrote info guide Getting Started and iPhone tabs with clear step-by-step setup and new-device connection instructions. Created this backlog document.
