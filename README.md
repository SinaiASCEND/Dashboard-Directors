# Alphabetize module sessions; remove week designation

## What this changes

Inside each module, sessions were grouped under **"Week 1 / Week 2 / …"** headers using a `week` field from source data. That field doesn't reflect the live academic calendar, so every time the real schedule shifted, the module view looked wrong.

This patch removes week-based scaffolding from list views:

1. **Desktop module session list** — sessions sorted A–Z by title; the "Week N" group headers are gone.
2. **Mobile module session list** — sessions sorted A–Z; the left-rail "Week N" column is gone.
3. **Subtitles everywhere** — "Week N" / "Wk N" stripped from every session sub-line, breadcrumb, and list eyebrow, so nothing in a list view can be mistaken for a schedule.

### Intentionally **not** changed

These are data fields on detail screens or user-driven filters, not list display, so they stay:

- `StatTile label="Week"` on the session detail card
- `DetailRow label="Week"` inside the session detail body
- The "Week" facet dropdown in global search filters
- The `"Week"` column in the alignment-audit CSV export

## How to apply

### Option A — `git apply` (preferred)

From the repo root:

```bash
git checkout -b sessions-alphabetical
git apply --check sessions-alphabetical.patch   # dry-run; should print nothing
git apply sessions-alphabetical.patch
git add index.html
git commit -m "Sort module sessions alphabetically; remove week designation from list views"
git push -u origin sessions-alphabetical
```

Then open a PR on GitHub.

### Option B — `git am` (preserves commit message)

```bash
git checkout -b sessions-alphabetical
git am sessions-alphabetical.patch
git push -u origin sessions-alphabetical
```

### Option C — Manual paste fallback

If `git apply` rejects any hunks (it sometimes does on a file that's been edited a lot since these snippets were extracted), use `MANUAL_EDITS.md` in this folder. It has every find/replace block as plain text — paste into your editor's find-replace and you're done.

## Sanity checks after applying

- Open any module on desktop → session list shows one alphabetized list, no week headers.
- Open any module on mobile → no week column on the left side of each row, sessions sorted A–Z.
- Open a session from a search result, AOC list, Thread list, or MEPO drill-down → the sub-line under the title no longer says "Week N".
- Open the session detail card → the "Week" stat tile is still there (this is intentional — it's a data field, not a schedule claim).
