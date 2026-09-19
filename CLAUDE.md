# CLAUDE.md — John Hendrix Memorial Prayer Walk Website

This file tells Claude how to work on this repository. It is written for **volunteers**
who run their own copy of Claude (e.g. Claude Code) against a local clone of this project.
Read it fully before making changes.

---

## 1. About this project

The website for the **John Hendrix Memorial Prayer Walk**, a 501(c)(3) nonprofit in
Oak Ridge, Tennessee. It honors John Hendrix, the "Prophet of Oak Ridge," whose early
20th-century visions foretold the Manhattan Project. The site replaces an older
Weebly/iPage site.

- **Live (staging) site:** https://johnhendrixmemorial.pages.dev/
- **Repository:** https://github.com/galapagosfinch/johnhendrixmemorial
- **Target production domain:** `johnhendrixmemorial.com` (cutover is pending — do not touch DNS)
- **Project lead:** Steve Finch. Steve makes the final call on content, design, and deployment.

**The most important design constraint is volunteer maintainability.** Prefer the simplest,
most boring solution that a non-expert could understand and edit later. Do not introduce
frameworks, build steps, backends, or dependencies without asking first.

## 2. Stack

| Layer | Choice |
| :-- | :-- |
| Framework | Astro 4, **static output** (`output: 'static'`) |
| Styling | Tailwind CSS 3 (`tailwind.config.mjs` — note the `.mjs` extension) |
| Hosting | Cloudflare Pages (project `johnhendrixmemorial`), build → `dist/` |
| Donations | PayPal links (live); Stripe Payment Links planned (no account yet) |
| Video | YouTube embed, click-to-load |
| Icons / fonts | Tabler icons and Google Fonts, loaded from CDNs in `src/styles/global.css` |

There is **no backend** in Phase 1. Anything dynamic is delegated to third-party hosted
services. Cloudflare Workers/D1 are only candidates for later phases.

## 3. Getting started (for the volunteer)

Prerequisites: Git, and Node.js 18.17.1 or newer (Node 20 LTS recommended), which Astro 4 requires.

```bash
git clone https://github.com/galapagosfinch/johnhendrixmemorial.git
cd johnhendrixmemorial
npm install
npm run dev        # → http://localhost:4321
```

| Command | Purpose |
| :-- | :-- |
| `npm run dev` | Local dev server with hot reload |
| `npm run build` | Production build to `dist/` — **run this before you say a change is done** |
| `npm run preview` | Serve the production build locally |

There is no test suite or linter. `npm run build` succeeding, plus a visual check in the
browser at both phone and desktop widths, is the bar.

### Working with Claude on this repo

When the volunteer asks you for something, Claude should:

1. **Assume the volunteer may not be a developer.** Explain what you changed and why in plain
   language, and give exact commands when they need to run something.
2. **Make small, focused changes.** One task per branch. Do not refactor unrelated code.
3. **Read the file before editing it**, and match the surrounding style.
4. **Run `npm run build`** after changes and report the result honestly.
5. **Ask before deciding** anything about content, donations, domains, or design direction (see §7).

## 4. Git workflow

- Never commit directly to `main` and never force-push.
- Create a branch per task: `git checkout -b short-description`.
- Push the branch and open a pull request. Steve reviews and merges.
- Cloudflare Pages builds a preview deployment for branches pushed to the main repository,
  so reviewers can see the change live. If the volunteer doesn't have write access, use a
  fork and open the PR from there.
- Write clear commit messages describing what changed and why.
- Never commit secrets. `.env`, `.env.local`, and `.env.production` are git-ignored — keep
  it that way. This site currently needs no secrets.

## 5. Project structure

```
src/
├── data/
│   └── trail-markers.json   ← Trail marker content (8 markers). Edit content HERE, not in the page.
├── layouts/
│   └── Layout.astro         ← Global header, footer, mobile nav, sticky mobile action bar
├── pages/
│   ├── index.astro          ← Home: Meet John Hendrix (timeline + sticky video sidebar)
│   ├── prayerwalk.astro     ← Trail guide (marker accordion)
│   ├── books.astro          ← Book resources
│   ├── geocaching.astro     ← Geocaching (caches currently archived)
│   ├── find-us.astro        ← Directions and map
│   └── donate.astro         ← Donation tiers, PayPal (Stripe later)
└── styles/
    └── global.css           ← Fonts, base styles, reusable component classes

public/
├── images/                  ← All images, locally hosted (migrated off the old Weebly CDN)
└── .well-known/
    └── apple-developer-merchantid-domain-association   ← Apple Pay placeholder; leave alone until Stripe is set up
```

Documentation lives in three places, and each has a distinct job:

- **`DESIGN.md`** — architecture, decisions, and *rationale* (why things are the way they are).
- **`README.md`** — how-to guides, plus the **TODO list, which is the source of truth for
  what work remains and in what order**. Check it before starting, and update it when you
  finish a task.
- **This file** — instructions for Claude.

Do not duplicate TODO items into this file or DESIGN.md.

## 6. Design system

Historically themed: aged paper, forest green, creek blue, warm copper, dark ink.

**Colors** (defined in `tailwind.config.mjs`, use these names — never hard-code hex values):
`parchment`, `parchment-dark`, `forest` (#2D4A2D), `forest-light`, `creek`, `creek-light`,
`copper`, `copper-light`, `ink`, `ink-soft`, `stone`, `stone-light`.

**Fonts:** `font-display` (Playfair Display — headings), `font-body` (Source Serif 4 — prose),
`font-ui` (Inter — nav, buttons, labels).

**Icons:** Tabler icon webfont (`<i class="ti ti-...">`). Don't add a second icon set.

**Reusable classes** (in `src/styles/global.css`): `section-pad`, `section-container`,
`eyebrow`, `pullquote`, `btn-primary`, `btn-secondary`, `btn-donate`. Reuse these before
writing new utility combinations. If you need a new pattern that will repeat, add a class
to `global.css` rather than copy-pasting long utility strings.

**Layout notes:**
- `section-container` is `max-w-4xl` — too narrow for two-column layouts. Widen *selectively*
  to `max-w-6xl` where needed (the homepage timeline does this).
- Design mobile-first. Many visitors will be standing on the trail using a phone.

### Accessibility requirements (WCAG 2.1 AA is the standard)

- Body text at least 16px; interactive targets at least 48×48px.
- Every image needs meaningful `alt` text; give images explicit `width` and `height` and use
  `loading="lazy"` below the fold.
- Maintain AA color contrast. Note that `stone` is weak on `parchment` — don't use it for
  body-sized text. Prefer `ink` / `ink-soft`.
- Use logical heading order and descriptive link text.
- Video needs captions or a transcript.

## 7. Rules and guardrails

### Ask Steve first (do not change on your own)

- **Payment and donation links** in `donate.astro` (PayPal URLs, the `stripeUrl` placeholder).
  Wrong links mean lost or misdirected donations.
- The contents of `public/.well-known/apple-developer-merchantid-domain-association`.
- **DNS, domain, or Cloudflare settings.** The custom-domain move is in progress and blocked
  by an account mismatch; leave it to Steve.
- `wrangler.toml` and `astro.config.mjs`.
- Adding npm dependencies, a backend, Workers, or any tool that increases maintenance burden.
- Anything that changes how volunteers must work (build tooling, folder layout).

### Historical and factual content — never invent

This is a memorial and history site about a real person and real events. **Do not make up,
embellish, or "improve" historical claims, quotes, dates, or testimonials.**

- Trail marker text must match the **physical signage** on the trail. If you are unsure whether
  text is authoritative, say so and ask rather than rewriting it.
- Do not fabricate geocacher testimonials or attribute quotes to real people.
- Keep a respectful, warm tone. The subject has a religious dimension (prayer, visions);
  represent it plainly and don't editorialize.

### Configuration gotchas (already learned the hard way)

- **Do not add a Cloudflare adapter to `astro.config.mjs`.** The site is fully static and
  needs none; with it present, local `npm run dev` breaks. (`@astrojs/cloudflare` remains in
  `package.json` but is unused.)
- **Downloading images from the old Weebly server:** use a bare `curl -o file URL`. Extra flags
  make the server return an HTML error page instead of the image, which gets saved as a
  "broken" file. Verify downloaded images actually open.
- **Fetching raw GitHub files** with web tools can be unreliable; use
  `curl -s https://raw.githubusercontent.com/galapagosfinch/johnhendrixmemorial/main/<path>`.

### Editing conventions

- **Hide sections by commenting them out, not deleting them.** This makes re-enabling
  straightforward. Currently commented out on purpose: the geocacher testimonials section
  (`geocaching.astro`), and the "Along the Trail" accordion and its `trailMarkers` import
  (`prayerwalk.astro`) unless the README says it has been re-enabled. Don't un-comment
  these unless asked.
- **Keep content out of layout code.** Text that volunteers will update should live in data
  files (like `trail-markers.json`) or clearly marked content blocks, not tangled into markup.
- Keep Astro components and pages simple and HTML-like. No client-side JavaScript unless it's
  necessary (accordion, click-to-load video/map, donation tier selector); if you add some,
  keep it small and inline, and mention it.
- Images: compressed JPEG in `public/images/`, lazy-loaded, with explicit dimensions.
- Don't pin CDN assets to `@latest` in new code; use a specific version.

## 8. Current status and where to look

Phase 1 (static site: six pages, design system, Cloudflare Pages deployment, PayPal donation
selector) is substantially complete. Remaining launch work, the Stripe/Apple Pay setup, geocaching
re-activation, and the later phases are tracked in the **README TODO list** — read it rather
than relying on any summary here, since it changes.

Before starting any task:

1. Read the relevant section of `README.md` (TODO) and `DESIGN.md` (rationale).
2. Confirm with the volunteer which task they are picking up, so two people don't collide.
3. When done, update the README TODO (and DESIGN.md only if a *decision* changed).

## 9. Definition of done

- [ ] `npm run build` passes with no errors
- [ ] Checked visually at phone width (~375px) and desktop width
- [ ] Meets the accessibility requirements in §6 (alt text, contrast, tap targets)
- [ ] No secrets, no hard-coded colors, no unrelated edits in the diff
- [ ] README TODO updated if applicable
- [ ] Branch pushed and PR opened with a plain-language description of the change
