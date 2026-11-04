# AGENTS.md — DnD Club Party Prep

Instructions for AI coding agents (and future-us) working on this project.

## What this is

A single-page party planner for a middle-school DnD club: checklist with notes,
comments on everything, guest list with RSVPs, a share button, a secret text-color
menu, winter snow, and public-domain Christmas music synthesized in the browser.

- **Live:** https://secretpizza14.github.io/dnd-club-party-prep/
- **Repo:** https://github.com/secretpizza14/dnd-club-party-prep
- **Canonical source:** `index.html` in this repo.
- A convenience copy lives at `~/Desktop/DnD Club Party Prep/index.html` so Levi can
  double-click it. The repo is the source of truth — after editing, copy repo → Desktop
  so the two never drift.

## Hard constraints

- **One self-contained `index.html`.** All CSS, JS, and the font are inline. No build
  step, no npm, no bundler, no external CSS/JS/font files.
- **No Google, no Firebase.** Levi explicitly does not want Google involvement. Do not
  add third-party services, accounts, analytics, or trackers without asking first.
- **Never commit secrets.** The site is public. No API keys with write power, passwords,
  or tokens in the page or repo.
- **Keep it working offline.** Opening the file directly must still work.
- **Keep the theme.** Denim/red/green/cream Christmas palette, pixel headings, plain
  sans body text. Don't redesign without being asked.
- **Respect `prefers-reduced-motion`** (snow is hidden when it's on).
- **Escape all user text** with `esc()` before inserting into HTML.

## Stack

- Vanilla HTML/CSS/JS, one file.
- ByteBounce (Medium) embedded as a base64 `@font-face`; headings/buttons use
  `var(--pixel)`, body copy uses system sans.
- No framework. Rendering is manual: mutate `state`, call `render()`, which calls `save()`.
- Persistence today is `localStorage` (per browser, not shared).

## Data model

`state` is stored under `localStorage["dnd-party-prep-v1"]`:

```js
{
  items: [{
    id, name, done,
    notes: [{ id, text, comments: [], open? }],
    comments: []
  }],
  guests: [{ id, name, rsvp: "going" | "maybe" | "no" }],
  comments: [] // party-level
}
```

Comments are `{ id, who, text, at }`. IDs come from `uid()`.

Other keys:

- `dnd-party-name` — remembered display name
- `dnd-text-color` — saved `--ink` color from the secret menu

`state.guests` and missing `notes`/`comments` arrays are repaired on load; keep that
pattern when adding fields so old saved data never crashes the page.

## Feature map

All in `index.html`:

- `seed()` — first-run data
- `commentThread(list, target)` — shared comment UI; targets are `party`,
  `item:<id>`, `note:<itemId>:<noteId>`
- `render()` — rebuilds items, guests, party comments; updates counts; saves
- delegated `submit` / `click` / `change` listeners — all actions use
  `data-action="..."` attributes
- `Music` — Web Audio carol sequencer (Jingle Bells, We Wish You a Merry Christmas,
  Deck the Halls). No audio files. Starts on first user gesture.
- Snow — 28 spans in `#snow`, `z-index:0` so it sits behind the panels
- Secret shortcut — typing `colcolcol` outside an input opens the text-color dialog
- `toast()` — bottom status popup
- Send link — `navigator.share` if available, otherwise clipboard, with a selectable
  fallback field

## Local testing

No Node on this machine. Use headless Chrome + JXA.

**JS syntax check:**

```sh
python3 - <<'PY'
import re
src = open('index.html').read()
open('/tmp/app.js','w').write(re.findall(r'<script>(.*?)</script>', src, re.S)[-1])
PY
osascript -l JavaScript -e 'ObjC.import("Foundation"); var s=$.NSString.stringWithContentsOfFileEncodingError("/tmp/app.js",$.NSUTF8StringEncoding,null).js; try{new Function(s);"PARSE OK"}catch(e){"PARSE ERROR: "+e.message}'
```

**Render / interaction test:** copy `index.html` to `/tmp/test.html`, append a
`<script>` that drives the page (submit forms, click buttons) and writes results into
a `<pre id="z">`, then:

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --virtual-time-budget=4000 \
  --dump-dom "file:///tmp/test.html"
```

**Screenshot:**

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --hide-scrollbars \
  --virtual-time-budget=2500 --window-size=1100,900 \
  --screenshot=/tmp/shot.png "file:///Users/levi/Projects/dnd-club-party-prep/index.html"
```

Headless Chrome enforces a ~500px minimum window width. To check phone widths,
load the page in 360px/320px iframes from a wrapper page and screenshot that.

## Deploy

GitHub Pages serves `main` / root. After editing:

```sh
cd ~/Projects/dnd-club-party-prep
cp "/Users/levi/Desktop/DnD Club Party Prep/index.html" index.html   # if edited on Desktop
git add -A
git commit -m "…"
git push
```

Pages rebuilds in ~40s. Verify the deployed copy, don't assume:

```sh
curl -s https://secretpizza14.github.io/dnd-club-party-prep/ | grep -c "your marker"
```

## Conventions

- 2-space indent, semicolons, single quotes in JS, template literals for markup.
- Keep functions small; prefer the existing delegated-event pattern over inline handlers.
- CSS lives in one `<style>` block, grouped by section with `/* ---------- */` headers.
- Colors come from CSS variables (`--red`, `--green`, `--ink`, …); don't hardcode
  duplicates.
- New UI should reuse `.panel`, `.addrow`, `.btn`, `.cbtn`, `.iconbtn`, `.empty`.
- Comments in code explain *why*, not *what*.
- Do not "clean up" the embedded base64 font — apps that ignore embedded fonts need
  the text outlined, but this page relies on the base64 face.

## Known gaps

- **No accounts yet.** Everything is per-browser; friends do not share a list.
- **No shared database.** A static GitHub Pages site cannot read a file on Levi's
  computer. Real accounts/shared data need a server or hosted backend.
- **No data retention/pruning.** Do not auto-delete user content; if pruning is ever
  added, make it explicit and reversible.
- The send button warns that a `file://` link only opens on that laptop; once hosted
  on Pages the link is real.

## When in doubt

Ask before: adding a dependency, adding a backend/service, changing the visual theme,
or deleting user data. Prefer working, tested, one-file changes over rewrites.
