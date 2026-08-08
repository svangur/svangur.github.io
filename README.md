# Svangur

A personal recipe site ("svangur" — Icelandic for *hungry*).

A static recipe site built with [Hugo](https://gohugo.io/) and the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, extended with a
custom recipe layer: cook-time stats, ingredient checklists (flat or grouped),
step photos with automatic resizing, recipe cross-reference cards,
ingredient-aware search, and schema.org Recipe markup for Google rich results.

Designed to deploy to GitHub Pages via GitHub Actions — every push to `main`
builds and publishes automatically.

## Prerequisites

| Tool | Why | Install |
|---|---|---|
| Hugo (extended) | The static site generator — the only hard dependency | `brew install hugo` |
| Git | Version control + theme submodule | preinstalled on macOS |
| ImageMagick | Optional — only used to generate placeholder cover images | `brew install imagemagick` |

The site was built against Hugo `0.164.0`. The deploy workflow pins the same
version in `.github/workflows/hugo.yaml` (`HUGO_VERSION`) — keep the two in
sync when upgrading.

## Local development

```sh
hugo server -D --disableFastRender
```

Then open <http://localhost:1313>. The site live-reloads on every save.

- `-D` includes draft recipes (`draft: true`), which are **excluded from
  production builds** — the classic "works locally, missing from the live
  site" gotcha.
- `--disableFastRender` matters here: without it, Hugo's dev server serves a
  stale search index (`/index.json`) after content edits. Production builds
  are unaffected either way.
- If a **deleted** page still serves locally (e.g. after removing a taxonomy
  or recipe): this Hugo version serves the dev site from `public/`, and
  builds never delete obsolete files there. Stop the server, `rm -rf public
  resources`, and restart. CI deploys are unaffected (fresh checkout).

## Writing a recipe

```sh
hugo new content recipes/beef-tacos
```

### Naming conventions

- **Folder slug**: always the *English* name of the dish, ASCII kebab-case
  (`beef-tacos`, not `bøf-tacos` or `BeefTacos`) — slugs become URLs. The
  `title` may be the dish's proper/native name; body text is English.
- **Cover photo**: named exactly after the folder — `beef-tacos/beef-tacos.jpg`
  — so a downloaded or shared image identifies its recipe.
- **All other images**: `<slug>-<descriptor>.jpg`, where the descriptor says
  what's *shown* (`beef-tacos-searing.jpg`), never step numbers — steps
  renumber when recipes get edited.
- **Tags**: lowercase, hyphenated when multiword (`meal-prep`); reuse the
  existing vocabulary before inventing a new tag.

This creates `content/recipes/beef-tacos/` from the archetype
(`archetypes/recipes/index.md`) as a *page bundle*: the recipe's `index.md`
and all its images live together in that folder.

### Front matter reference

```yaml
title: Beef Tacos
date: 2026-08-03
draft: true            # flip to false (or delete) to publish
description: One-line intro shown under the title and used for SEO.
summary: List-page teaser — usually same as description. If omitted, lists
  show the start of the body (i.e. instruction text), which you don't want.
prepTime: 15           # minutes — feeds the stats bar and schema.org
cookTime: 30           # minutes
servings: 4
nutrition:             # optional — renders a Nutrition Facts card and feeds
  calories: 350        # schema.org; per serving, grams (sodium in mg), any
  protein: 20          # subset of: calories, protein, carbs, fat, fiber,
  carbs: 40            # sugar, sodium
  fat: 12
tags: [dinner, mexican, quick] # the single taxonomy: meal, cuisine, effort,
                               # diet… — powers the home page chips + search
cover:
  image: beef-tacos.jpg # cover photo, named exactly after the folder
  alt: Tacos on a board
ingredients:
  - 500 g ground beef
  - 12 corn tortillas
```

Ingredients can instead be **grouped into sections** (don't mix the two forms
in one recipe):

```yaml
ingredients:
  - name: Filling
    items:
      - 500 g ground beef
      - 1 onion, diced
  - name: Salsa
    items:
      - 3 tomatoes
```

### Body convention

The Markdown body holds only `## Instructions` and (optionally) `## Tips` —
the intro belongs in `description`, ingredients in front matter. Instructions
can be split into sections with `###` subheadings; numbering restarts per
section:

```markdown
## Instructions

### Filling

1. Brown the beef...

### Salsa

1. Dice the tomatoes...
```

### Step photos

Drop the image file in the recipe's folder and reference it under a step,
indented three spaces with a blank line above (this keeps it attached to the
step and preserves list numbering):

```markdown
2. Cook until bubbles form on the surface.

   ![Bubbles forming](beef-tacos-searing.jpg "Optional caption — when it looks like this, flip.")

3. Flip and cook the other side.
```

Full-size phone photos are safe to drop in as-is: anything wider than 720px is
automatically served as resized 720/1440px variants via `srcset`. The quoted
title becomes a caption. `{width=400}` after the image works for a smaller
inline size.

### Cover images

Any landscape photo named to match `cover.image` (convention: `<slug>.jpg`),
ideally ≥1200px wide. It appears as the list-page thumbnail, the hero on the
recipe page, in social preview cards, and in the schema.org markup. In
production, PaperMod generates five responsive sizes automatically.

To regenerate the placeholder gradients used by the sample recipes, see the
ImageMagick pattern in the repo history — real photos should replace them.

### Meal prep / batch recipes

Still regular recipes — an optional `mealprep` block adds a
"Storage & Reheating" card in the left column, under the ingredients (all
fields free-text, any subset):

```yaml
mealprep:
  yields: 8 portions / 4 containers
  fridge: 4 days
  freezer: 3 months
  reheat: Microwave 2–3 min, covered.
```

Recipes without the block are untouched. Tag batch recipes `meal-prep` so the
home page chip can isolate them.

### Component recipes (dough, sauces, stocks)

Also regular recipes — no special type. Conventions:

- A `components` tag groups them — one chip click on the home page isolates
  them (or, with other chips, excludes them).
- Reference them from recipes that use them with the `recipe` shortcode
  ("You'll need one batch of…").

Rule of thumb for future content: **if it has ingredients and instructions,
it's a recipe** — model variations as optional front matter, not new content
types. New sections are for genuinely different shapes of content.

### Dynamic servings & macros

Recipe pages have two client-side steppers (the site's only JavaScript
feature; without JS the static base values show):

- **Servings** (next to "Serves:" in the hero): − / + buttons, or type any
  value directly — decimals work, so 4.5 servings of a 3-serving recipe is a
  1.5× batch. Scales every ingredient line's *leading* quantity — plain
  numbers, decimals, `1/2`-style fractions, and `2-3` ranges. Lines with no
  leading number ("Pinch of salt", "to taste") are untouched, so write
  quantities at the start of the line.
- **Eating N portion(s)** (nutrition tab): a personal multiplier — label
  values are per-portion macros × N, so eating 1.5 portions shows 1.5× the
  numbers. Independent of the Serves control (scaling a batch never changes
  what's in one portion); the label's "whole batch" line is what tracks
  Serves.

Scaled values are display-only — the schema.org markup always carries the
recipe's base values.

### Migrating a recipe collection (checklist for humans & AI assistants)

When converting recipes in bulk from another source (blog export, notes,
another app), each recipe must become exactly this:

```
content/recipes/<kebab-case-slug>/
  index.md     ← required
  <slug>.jpg   ← cover, optional (cards show a blank panel without it)
  *.jpg        ← optional step photos, referenced from index.md
```

**Front matter — canonical template.** `title`, `date`, `description`,
`summary`, `servings`, `prepTime`, `cookTime`, `tags`, and `ingredients` are
expected on every recipe; the rest is optional:

```yaml
---
title: Beef Tacos                # required
date: 2019-04-12                 # original creation date if known — this
                                 #   drives the "Newest" sort; fall back to
                                 #   the migration date
draft: true                      # keep true until reviewed, then remove
description: One-line intro shown under the title.
summary: Usually identical to description (list-page teaser).
prepTime: 15                     # minutes, integer
cookTime: 30                     # minutes, integer
servings: 4                      # integer — powers the scaler
tags: [dinner, mexican, quick]   # lowercase; REUSE the existing vocabulary
                                 #   (ls content/recipes/*/index.md | check
                                 #   tags, or see /tags/) — don't invent
                                 #   synonyms like "veggie" vs "vegetarian"
cover: { image: beef-tacos.jpg, alt: Tacos on a board }  # only if photo exists
ingredients:                     # flat list, or grouped (see above) — NEVER
  - 500 g ground beef            #   in the body. Quantity FIRST on the line
  - 12 corn tortillas            #   ("500 g beef", not "beef, 500 g") — the
  - Salt, to taste               #   scaler parses the leading number
nutrition: { calories: 350, protein: 20, carbs: 40, fat: 12 }  # optional
mealprep: { fridge: 4 days, reheat: … }                        # optional
---
```

**Body — only these two sections**, nothing else (no ingredient lists, no
title heading, no metadata):

```markdown
## Instructions

1. Step text…            ← ordered lists render as "Step N"
2. …

## Tips                   ← optional

- …
```

Use `### Subsection` headings inside Instructions for multi-part recipes.
Internal links must use `relref` or the `recipe` shortcode (checked at build
time) — never plain `/recipes/...` markdown links.

**Validate after migrating:** `hugo` must build with zero errors (broken
cross-references and malformed front matter fail loudly), then spot-check
`hugo server -D --disableFastRender` — the home grid, one migrated recipe
page (scaler works ⇒ ingredients parse), and the tag chips (an exploded tag
vocabulary means synonyms slipped in).

### Referencing another recipe

Two forms, both **build-time checked** — a typo'd or deleted target fails the
build instead of shipping a dead link:

```markdown
Serve with [Spaghetti Aglio e Olio]({{< relref "/recipes/spaghetti-aglio-e-olio" >}}).
```

```markdown
## See Also

{{< recipe "spaghetti-aglio-e-olio" >}}
```

The `recipe` shortcode renders a linked card with the target's thumbnail,
description, and total time — all pulled live from the target, so it stays
current if that recipe changes. Avoid plain `[text](/recipes/...)` links for
internal references; they get no build-time check.

## Home page

The home page is the site's single browse surface: the weekly meal plan on
top, then the full recipe grid with search, multi-select tag chips, and
sorting (newest first by default — that's also the no-JS fallback).
`/recipes/` redirects to the home page so breadcrumbs and old links keep
working; there is no nav menu — everything is reachable from here.

### Weekly meal plan

The "This week's plan" strip is driven by `data/mealplan.yaml`:

```yaml
weeks:
  - start: "2026-08-03"   # quoted Monday date
    days:                  # any subset of monday..sunday
      monday: spaghetti-aglio-e-olio
      wednesday: chicken-fried-rice
```

- Day values are recipe folder names; a typo **fails the build**.
- The template picks the week whose 7-day window contains the *build* date.
  The deploy workflow has a Monday-morning cron trigger, so queueing up
  future weeks in the file makes them appear on schedule with no push.
  (GitHub pauses cron workflows after ~60 days without repo activity; any
  push re-enables them.)
- The current weekday is highlighted client-side, so it stays correct between
  weekly builds. Weeks with no matching entry render nothing.

## Search & filtering

Search lives on the home page (there is no separate search
page): a text input runs Fuse.js over the build-time index
(`/index.json`), which covers title, description, **ingredients** (flattened
from either form), tags, and instruction text — so searching an ingredient
works even if it's never mentioned in the body. Alongside it, the tag chips
are multi-select with AND semantics (click to add, click again to remove) —
"dinner ✕ quick" shows only quick dinners — and both combine with the sort
dropdown.

Fuse tuning lives in `hugo.toml` under `[params.fuseOpts]`: keys are weighted
(title highest, ingredients next, body text lowest) and the fuzziness
threshold is tightened from the Fuse default 0.4 to 0.25 — loose enough for
typos ("chiken" finds Chicken), tight enough that "chick" doesn't match
"whisk". For strictly literal search, remove the `content` entry from
`fuseOpts.keys`.

## Design system

Both color modes are from the [Ayu](https://github.com/ayu-theme) family:
**Ayu Light** in light mode, **Ayu Mirage** in dark. All of it lives in
`assets/css/extended/recipes.css` as CSS-variable overrides of PaperMod's
palette; the theme's own files are untouched. Conventions:

- `--accent` (`#ffaa33` light / `#ffcc66` dark) marks every interactive
  highlight: active tabs and filter chips, checked checkboxes, today's meal
  plan cell, hover borders, keyboard focus outlines. `--accent-strong` is the
  accent used *as text* (darkened in light mode for contrast).
- `--card-radius` (12px) for all card surfaces; 8px for cells/thumbnails
  inside cards; 5–6px for controls (chips, steppers, inputs).
- Surface layering: page `--theme` → cards `--entry` → sunken panels inside
  cards `--code-bg`.
- One deliberate exception: the light-mode Nutrition Facts label stays
  black-on-white like a printed label; dark mode restyles it natively.

## Discoverability — intentionally unlisted

The site is public at its URL but hardened against being *found*:

- `robots.txt` disallows all crawling (`layouts/robots.txt` shadows the
  theme's crawl-friendly version).
- Every page carries `<meta name="robots" content="noindex, nofollow">`,
  applied site-wide by setting PaperMod's `robotsNoIndex` page param through
  a `[[cascade]]` block in `hugo.toml`.
- RSS feeds and the sitemap are disabled entirely
  (`disableKinds = ['rss', 'sitemap']`), so there is nothing to subscribe to
  or crawl. The site's own search is unaffected.

This is obscurity, not access control: anyone with the link can view the
site. To make it publicly discoverable someday, remove `disableKinds`, the
cascade block, and `layouts/robots.txt`, and restore `'RSS'` to
`outputs.home`.

## SEO

Every recipe page emits [schema.org Recipe](https://schema.org/Recipe) JSON-LD
built from the front matter: times (ISO 8601 durations), yield, calories,
category, keywords, flattened ingredients, instructions text, and the
cover image URL. While the site is unlisted this is dormant, but if it ever
goes public it makes recipes eligible for Google's rich results (photo +
rating + cook time cards). Validate with Google's
[Rich Results Test](https://search.google.com/test/rich-results) if enabled.

## Project structure — what's custom and why

Everything below is ours; the theme itself stays a pristine, updatable copy in
`themes/PaperMod/` (Hugo's lookup order means our files shadow the theme's).

| Path | Purpose |
|---|---|
| `hugo.toml` | Site config: taxonomies, menus, search tuning, image-attribute markup opts |
| `archetypes/recipes/index.md` | Template for `hugo new content recipes/<slug>` — documents all fields inline |
| `data/mealplan.yaml` | The weekly meal plan shown on the home page |
| `layouts/recipes/single.html` | Recipe page layout: split hero (title/meta/photo), then a sticky ingredients card beside the method column with "Step N" numbering. Originally derived from the theme's `single.html`, now intentionally divergent — it only shares the theme's partials |
| `layouts/_partials/recipe_meta.html` | Hero meta lines (serves/time/calories) + linked taxonomy chips |
| `layouts/home.html` | The single browse surface: meal plan + Fuse.js search + multi-select tag chips (AND) + sort over the full card grid |
| `layouts/_partials/meal_plan.html` | Renders the current week from `data/mealplan.yaml` |
| `layouts/_partials/recipe_card.html` | Compact photo card used in the home grid |
| `layouts/recipes/section.html` | Redirects /recipes/ to the home page |
| `layouts/_partials/recipe_ingredients.html` | Ingredient checklist card; renders flat or grouped form |
| `layouts/_partials/recipe_schema.html` | schema.org Recipe JSON-LD |
| `layouts/_partials/recipe_nutrition.html` | Nutrition Facts label, shown in the ingredient card's "Nutrition" tab (CSS-only tabs, no JS) |
| `layouts/_partials/recipe_mealprep.html` | Storage & Reheating card (left column) rendered from `mealprep` front matter |
| `layouts/_partials/extend_head.html` | Injects the schema partial on recipe pages (PaperMod's designated hook) |
| `layouts/_partials/functions/flat-ingredients.html` | Shared helper: flattens grouped ingredients for schema + search |
| `layouts/_markup/render-image.html` | Body-image render hook: bundle resolution, auto-resize + `srcset`, `<figure>` captions |
| `layouts/_shortcodes/recipe.html` | The `{{< recipe "slug" >}}` cross-reference card |
| `layouts/index.json` | Search index; extends the theme's version with recipe fields |
| `layouts/robots.txt` | Deny-all robots.txt (site is intentionally unlisted) |
| `assets/css/extended/recipes.css` | All recipe styling + print stylesheet (PaperMod's designated CSS extension point) |
| `.github/workflows/hugo.yaml` | Build & deploy to GitHub Pages on push to `main` |

Generated directories (`public/`, `resources/`) are gitignored — CI rebuilds
them on every deploy.

## Deploying to GitHub Pages

One-time setup (not yet done for this repo):

1. `git init` — then convert the theme to a submodule so CI can fetch it:

   ```sh
   rm -rf themes/PaperMod
   git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
   ```

2. Commit everything and create the GitHub repo (`gh repo create`), push to
   `main`.
3. In the repo: **Settings → Pages → Build and deployment → Source →
   "GitHub Actions"**.

From then on every push to `main` deploys in about a minute. The workflow
injects the correct `baseURL` for the Pages URL automatically — the
placeholder in `hugo.toml` is only a fallback. Remember: recipes still marked
`draft: true` will not appear on the live site.

## Possible next steps

- **Multilingual**: Hugo + PaperMod fully support it (PaperMod ships UI
  translations for 46 languages, incl. Danish; page bundles let translations
  share photos). One prerequisite: the English labels hardcoded in the recipe
  partials ("Ingredients", "Prep", "Cook", …) need moving to Hugo's `i18n`
  files first — cheap now, tedious after fifty recipes.
- **Card-grid recipe index**: list pages are currently PaperMod's vertical
  feed; a photo-grid layout means overriding one list template plus CSS.
- **Real cover photos** to replace the generated gradient placeholders.
