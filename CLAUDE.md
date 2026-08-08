# Svangur — agent handover

You are picking up a **finished-but-undeployed** personal recipe site. This
file is the state-and-goals handover; **README.md is the full manual** —
authoring spec, migration checklist, naming conventions, design system,
deploy steps, dev-server quirks. Read it before changing anything.

## Hard constraints (non-negotiable)

- **Static site only.** Hugo-generated files, no server-side anything. All
  interactivity is client-side JS (search/filter/sort, servings scaler,
  portions multiplier) over build-time data (`/index.json`).
- **Deploys to GitHub Pages** via the included workflow
  (`.github/workflows/hugo.yaml`): push to `main` → build → publish. A Monday
  cron rebuild keeps the home-page meal plan current.
- **Self-contained**: no CDNs, no external fonts/scripts.
- **Intentionally unlisted**: noindex on every page, deny-all robots.txt, no
  RSS/sitemap. Keep it that way unless the user says otherwise.

## Current state

- Feature-complete, design-audited, production build clean (zero errors, no
  broken internal links). **Never committed to git** — no repo exists yet.
- `themes/PaperMod/` is a plain copy (its `.git` was stripped for the zip) —
  it must become a **submodule** during deploy, or CI can't fetch it.
- Hugo 0.164.0 locally; the workflow pins the same (keep in sync).
- Four **sample recipes** (classic-american-pancakes, spaghetti-aglio-e-olio,
  chicken-fried-rice, steamed-rice) with generated gradient placeholder
  covers. They demonstrate every feature (grouped ingredients, step photos,
  meal prep card, components pattern, cross-references).

## Priority 1 — deploy to GitHub Pages

The user will log into a **personal** GitHub account (not their work
account) via `gh auth login` and give you the username. Then:

```sh
cd <project>
git init
git config user.name "<name the user chooses>"
git config user.email "<personal or GitHub-noreply email>"  # NOT the work email
rm -rf themes/PaperMod
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git add -A && git commit -m "Svangur: initial site"
gh repo create <name> --public --source=. --push   # public: free-plan Pages requires it
gh api -X POST repos/<user>/<name>/pages -f build_type=workflow  # Pages source = Actions
```

Verify: Actions run green → site serves at `https://<user>.github.io/<name>/`
→ spot-check a recipe page (scaler works), search, and robots.txt
(`Disallow: /`). Note: `data/mealplan.yaml`'s only week starts 2026-08-03 —
if that week has passed, the plan strip won't render until a current week is
added (quoted Monday date; a typo'd recipe slug **fails the build**).

## Priority 2 — migrate the user's recipe collection

- Source: the user's existing recipes, **already in markdown** (they will
  provide the files). Format details unknown — assess, then map to the spec.
- Follow README **"Migrating a recipe collection"** exactly: one folder per
  recipe, English kebab-case slugs, cover named `<slug>.jpg`, other images
  `<slug>-<descriptor>.jpg`, quantities first on ingredient lines, lowercase
  tags **reusing the existing vocabulary**, `summary` ≈ `description`,
  original dates if known.
- Validate: `hugo` builds clean; scaler parses ingredients on a sample; tag
  chips didn't sprout synonyms.
- **Afterwards, delete the four sample recipes** (they are scaffolding, per
  the user) and update `data/mealplan.yaml` to reference real recipes —
  stale slugs there break the build.

## Things that bite (learned the hard way)

- **Dev server serves from `public/` on disk and never deletes obsolete
  files** — deleted pages keep serving until `rm -rf public resources` +
  restart. Always run with `--disableFastRender` (stale `/index.json`
  otherwise). Details in README → Local development.
- **Theme coupling**: custom layouts use PaperMod partials and class names
  (e.g. `.header-nav`) — after a theme update, eyeball a recipe page and the
  home page.
- Screenshots via headless Chrome on macOS lie below ~500px width (min
  window size) — use CDP `Emulation.setDeviceMetricsOverride` for mobile
  checks. A working script pattern exists; rebuild it if needed.
- Build-time safety is a feature: bad cross-references, bad meal-plan slugs,
  malformed front matter all **fail the build**. Treat build errors during
  migration as the validator doing its job.

## User preferences (observed over the build)

- **Less is more**: they chose to merge browse/search/home into one page and
  collapse taxonomies to a single `tags` field. Don't reintroduce structure
  without asking.
- Design is **Ayu Light / Ayu Mirage** with the gold/amber accent — the
  README's Design system section has the rules (`--accent`, `--card-radius`,
  surface layering). They rejected a pixel font (Spleen) after trying it.
- They appreciate honest trade-off explanations before structural changes,
  and verified results (real builds, real screenshots) over claims.
