# John Hendrix Memorial Prayer Walk — Website

Built with [Astro](https://astro.build) and [Tailwind CSS](https://tailwindcss.com), deployed on [Cloudflare Pages](https://pages.cloudflare.com).

For the architecture, design rules, and the reasoning behind them, see [DESIGN.md](DESIGN.md). This README covers day-to-day work: running the site, editing content, deploying, and what is left to do.

IMPORTANT: the majority of this code was written with the free version of Claude using Sonnet 5 Medium.  The free version can generate snippets as well full file replacements that can be downloaded or copy/pasted.  You should be able to use `CLAUDE.md` to initialize a future version of Claude.

## Getting Started

```bash
npm install
npm run dev
```

Open http://localhost:4321 in your browser.

## Project Structure

```
src/
├── data/
│   └── trail-markers.json     ← Edit trail marker content here
├── layouts/
│   └── Layout.astro           ← Global header, footer, sticky mobile bar
├── pages/
│   ├── index.astro            ← Homepage (Meet John Hendrix)
│   ├── prayerwalk.astro       ← Prayer Walk trail guide
│   ├── books.astro            ← Books resource page
│   ├── geocaching.astro       ← Geocaching page (archived)
│   ├── find-us.astro          ← Directions and map
│   └── donate.astro           ← Donation page
└── styles/
    └── global.css             ← Shared component classes (buttons, layout, labels)

public/
├── images/                    ← All site photos live here
└── .well-known/
    └── apple-developer-merchantid-domain-association  ← Apple Pay domain file

tailwind.config.mjs            ← Colors, fonts, and type scale
DESIGN.md                      ← Architecture, design system, decisions
```

---

## Editing Content

Changing words, photos, links, or amounts should never require touching layout or styling code. Most content sits in a list at the top of its page (between the `---` lines) or in a JSON file.

### Where each kind of content lives

| To change... | Edit this |
| :--- | :--- |
| Trail marker titles, text, photos | `src/data/trail-markers.json` (each marker has `id`, `title`, `description`, `imageUrl`, `imageAlt`) |
| Photo carousel on the Prayer Walk page | The list of `src` / `alt` pairs near the top of the carousel in `src/pages/prayerwalk.astro` |
| Homepage timeline | The `<li>` entries in `src/pages/index.astro`. To add an entry, copy an existing `<li>` block and change the year, heading, and text. |
| Books (title, cover, description, price, Amazon link) | The `books` list at the top of `src/pages/books.astro` |
| Local retailers | The `retailers` list at the top of `src/pages/books.astro` |
| Donation tiers (amount, name, description) | The `tiers` list at the top of `src/pages/donate.astro` |
| PayPal link | The PayPal constants in `src/pages/donate.astro` (one at the top of the file, one in the script at the bottom; keep them in sync) |
| Geocache cards and testimonials | The `caches` and `testimonials` lists at the top of `src/pages/geocaching.astro` |
| Address, map link, coordinates | The `location` object at the top of `src/pages/find-us.astro` (the trail rules are in the page body) |
| Header, footer, mobile action bar | `src/layouts/Layout.astro` |
| Phone number | It appears in several places: `Layout.astro` (footer and mobile bar), `donate.astro`, and `find-us.astro`. Search the `src` folder for `712-8027` and `8657128027` and update every match. |

### Adding or replacing a photo

1. Save the image in `public/images/`. Keep the file size modest (compress it first if it is a large camera original).
2. Reference it as `/images/your-file.jpg`.
3. Always include `alt` text describing the image, plus `width` and `height` so the page does not jump while loading.
4. Use `loading="lazy"` for anything below the top of the page.

### Hiding a section without deleting it

Sections that are not ready are hidden by commenting them out, not deleting them, so bringing them back is a one-step change. Use HTML comments (`<!-- ... -->`) around markup, and comment out any data import that only that section uses. The geocacher testimonials section on `geocaching.astro` is an example: uncomment it and replace the placeholder quotes when real ones are collected.

---

## Design System Guide

The palette, fonts, and type scale are defined in `tailwind.config.mjs`. The shared component classes are in `src/styles/global.css`. The reasoning is in section 6 of [DESIGN.md](DESIGN.md).

### Which class to use for what

| Class | Use it for |
| :--- | :--- |
| `btn-primary` | Main action button (forest green) |
| `btn-secondary` | Secondary action button (outlined) |
| `btn-donate` | Giving actions only (copper) |
| `section-container` | Centers content at a readable width (`max-w-4xl`) |
| `section-pad` | Standard vertical spacing for a page section |
| `bg-forest-textured` | Dark green background for heroes and the header |
| `hero-border` | Copper accent line down the left edge of a hero |
| `eyebrow` | Small uppercase label above a heading, on light backgrounds |
| `eyebrow-light` | Same label for dark backgrounds |
| `copper-divider` | Copper gradient rule between blocks |
| `pullquote` | Highlighted quotation with a copper left border |
| `tier-active` | Selected state of a donation tier button |

Pages generally open with a `bg-forest-textured` hero, then alternate `bg-parchment` and `bg-parchment-dark` sections.

**Widening a section:** `section-container` is intentionally narrow (`max-w-4xl`). For a two-column layout, widen just that section to `max-w-6xl`, the way the homepage timeline does. Do not change `section-container` itself.

### Palette and contrast note

The goal is WCAG AA: at least 4.5:1 contrast for normal text (3:1 for large text). Safe pairings for small text:

* `ink`, `ink-soft`, or `forest` on `parchment` or `parchment-dark`
* `copper` on `parchment` (labels and accents)
* `parchment` or `parchment-dark` on `forest`, `forest-light`, or `copper`
* `stone-light` or `stone` on `ink` (footer)

Pairings to avoid for small, essential text until the fixes in the TODO below are done:

* `stone` on `parchment` or `parchment-dark` (about 3.5:1 and 3.1:1)
* `creek-light` on `forest` (about 3.6:1; this is what `eyebrow-light` uses)

---

## Deploying

The site is deployed by Cloudflare Pages, which builds automatically whenever changes are pushed to GitHub.

| Setting | Value |
| :--- | :--- |
| Repository | `galapagosfinch/johnhendrixmemorial` |
| Cloudflare Pages project | `johnhendrixmemorial` |
| Live address | https://johnhendrixmemorial.pages.dev |
| Build command | `npm run build` |
| Build output directory | `dist` |

### Previewing changes before they go live

Cloudflare Pages builds a separate preview for every branch, so you can review a change on its own before it reaches the live site.

1. Create or switch to a branch (for example `dev`) and push your changes.
2. In Cloudflare, open Workers & Pages, choose the `johnhendrixmemorial` project, and open the Deployments tab. The branch's preview deployment and its preview link appear there.
3. Review the preview on both a phone and a desktop browser.
4. When you are happy with it, merge the branch (for example through a pull request) into the main branch. Cloudflare then publishes it to the live site.

### Publishing to the live site

Merging to the main branch triggers a production build automatically. There is nothing else to run. The deployment status shows in the same Deployments tab.

### First-time setup (already done)

1. Push the repository to GitHub.
2. In Cloudflare Pages, choose Create a project and connect the Git repository.
3. Set the build command to `npm run build` and the output directory to `dist`.
4. Cloudflare handles SSL, the CDN, and deployment from then on.

---

## Completed

- Full site scaffold (6 pages: index, prayerwalk, books, geocaching, find-us, donate)
- Global layout with sticky mobile nav bar
- Design system (parchment/forest/creek/copper/ink palette, Tabler icons, Google Fonts)
- Images hosted locally in `public/images/`
- Homepage with timeline and sticky video sidebar
- Trail marker text filled in, with the marker accordion live on the Prayer Walk page
- Find Us page with updated coordinates and a click-to-load map
- Interactive donation tier selector with PayPal pre-fill
- Both geocache URLs added (GC5JYGG markers 1–4, GC5JYFR markers 5–8)
- Deployed to Cloudflare Pages

---

## Short-term TODO

### Donations
- **Pre-selected Donate buttons:** Create a new PayPal URL for each pre-selectable amount, then update `donate.astro` so each preset button uses its own URL.

### Domain and hosting
- **Custom domain:** Point `johnhendrixmemorial.com` to Cloudflare Pages (transfer initiated).
- **Resolve the Cloudflare account mismatch:** The DNS zone and the Pages project are in different Cloudflare accounts, which blocks attaching the domain.

### Content
- **Trail marker photos:** Add a photo for each of the 8 markers (`imageUrl` in `trail-markers.json` is currently empty for all of them).
- **Video captions:** Verify captions are enabled on the YouTube documentary, and decide whether to add a transcript link.

### Design and accessibility
- **Visual review:** Do a full review on mobile and desktop before launch.
- **Contrast fixes:** Darken the `stone` color token, and fix `eyebrow-light` (creek-light on forest) so small text reaches 4.5:1.
- **Static map image:** Replace the plain "Tap to load interactive map" panel on the Find Us page with a real static map image.

### Performance and SEO
- **Images:** Compress the existing images in `public/images/`, and add the missing `width` and `height` to the Prayer Walk marker and gravesite images and to the homepage video thumbnail.
- **Site assets:** Create `public/favicon.svg`, the default social sharing image at `public/images/og-default.jpg` (both are already referenced by `Layout.astro`), a `robots.txt`, and a sitemap.
- **Fonts and icons:** Pin the Tabler icons version (it currently loads `@latest`) and consider self-hosting fonts and icons.

---

## Phase 2

### Stripe/Apple Pay setup
The choice between Stripe Payment Links and Square Payment Links is still open (see section 4.1 of [DESIGN.md](DESIGN.md)). If Stripe:

1. Create a Stripe account and register your domain for Apple Pay.
2. In `src/pages/donate.astro`, add your Stripe Payment Link URL and a button for it. **Stripe account not yet created.**
3. Download the domain verification file from Stripe.
4. Replace the contents of `public/.well-known/apple-developer-merchantid-domain-association`.

### Geocaching
- Re-activate the geocaches (currently archived: GC5JYGG, GC5JYFR).
- In `src/pages/geocaching.astro`, uncomment the testimonials section and replace the placeholders with real quotes from geocachers.

---

## Phase 3

- Geocaching API integration via Cloudflare Workers
- Donation progress meter (Cloudflare D1)
