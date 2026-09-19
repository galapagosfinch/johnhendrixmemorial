# Technical Design Document: Website Rebuild on Cloudflare Pages

**Project:** John Hendrix Memorial Prayer Walk Digital Modernization  
**Architect:** Steve Finch  
**Last updated:** September 2026

---

This website represents the John Hendrix Memorial Prayer Walk in Oak Ridge, TN. It was rebuilt from scratch on a modern web stack with a focus on performance, accessibility, and user experience, and it is hosted on Cloudflare Pages. The majority of the site is static pages with text, images, and video. The Donation page uses PayPal's hosted checkout at launch; Stripe Payment Links (or Square Payment Links) is the planned addition in Phase 2. No card data ever touches this site.

Task tracking and day-to-day how-tos live in the [README](README.md). This document records architecture, design rules, and the reasoning behind them.

---

## 1. Architecture Overview

To move the legacy Weebly/iPage site to a modern, high-performance, mobile-friendly platform, the new site is built entirely on **Cloudflare Pages**.

Because the historical content is primarily static, a **Jamstack (JavaScript, API, and Markup)** architecture is the right fit. It eliminates server-side rendering, moves delivery to Cloudflare's global edge network, and keeps page loads fast even on spotty mobile data out on the physical nature trail.

Astro builds the site as fully static output (`output: 'static'`) and Cloudflare Pages serves the resulting `dist` folder. No Cloudflare adapter is needed for static output.

No serverless backend (Cloudflare Workers) is required for Phases 1 or 2. All dynamic functionality is delegated to hosted third-party services (PayPal, later Stripe or Square, Google Maps, YouTube), which keeps the architecture simple, maintainable by volunteers, and free of ongoing infrastructure overhead. Workers and D1 enter only in Phase 3.

---

## 2. Component Architecture Diagram

```
         +------------------------------------------+
         |            Cloudflare DNS / Edge         |
         +------------------------------------------+
                               |
                               v
                  +-----------------------+
                  |   Cloudflare Pages    |
                  |  (Static Assets & UI) |
                  +-----------------------+
                  |  - Astro (static)     |
                  |  - Tailwind CSS       |
                  +-----------------------+
                               |
                               v
                +----------------------------------+
                |       Third-Party Services       |
                +----------------------------------+
                | - PayPal Hosted Checkout         |
                | - Stripe Payment Links (planned) |
                | - Google Maps embed (keyless)    |
                | - YouTube embed (click-to-load)  |
                | - Geocaching.com (links)         |
                | - Google Fonts + Tabler icons    |
                |   (CDN, see section 4)           |
                +----------------------------------+
```

---

## 3. Structural Breakdown by Layout & Page Modules

The application is decomposed into a shared layout plus six pages built as Astro components. Page content that volunteers are likely to change (trail markers, book and retailer entries, donation tiers, geocache cards) is kept as plain data at the top of each page or in a JSON file, separate from markup.

### 3.1 Global Layout Component (Navigation & Sticky Elements)
* **Analysis:** All pages share an identical header, footer, and navigation.
* **Implementation:** A single `Layout.astro` provides a sticky header with a desktop nav (the Donate link is styled as a copper button) and a responsive hamburger menu on mobile. The layout also sets per-page title, description, and Open Graph tags.
* **Footer:** A global footer shows the 501(c)(3) tax-exempt status, the phone number, a Donate link, and copyright text.
* **Mobile action bar:** On mobile only, a fixed bottom bar puts *Call*, *Find Us*, and *Donate* within thumb reach (see 5.2).

### 3.2 Meet John Hendrix (Homepage)
* **Analysis:** Tells the historical narrative of John Hendrix ("The Prophet of Oak Ridge"), his 40 nights in the woods, and his Manhattan Project prophecies.
* **Implementation:** An `.astro` page with a hero, a dated timeline of his story, and a closing "Walk Where John Walked" call to action. On desktop the timeline sits in a left column beside a sticky sidebar holding the YouTube documentary and a call to action; this section widens to `max-w-6xl` to make room for two columns. On mobile the columns stack.
* **Video loading:** The video is a click-to-load facade. The page renders only a YouTube thumbnail, and the YouTube iframe is mounted when the visitor taps play. No YouTube code loads until then, which keeps the initial page light on trail data connections.

### 3.3 John Hendrix Memorial Prayer Walk (The Trail Guide)
* **Analysis:** Details a nature trail along Hendrix Creek with eight story markers, local flora and wildlife, train tracks, and a boxwood gravesite.
* **Implementation:** A hero, a photo carousel (seven images with previous/next buttons and dot indicators), a trail introduction, the eight markers, and a gravesite section.
* **Trail markers:** The eight markers live in a static JSON data file (`src/data/trail-markers.json`) and are rendered as a client-side accordion at every screen size, which reduces visual noise and lets on-site hikers open one marker at a time. Each entry has a title, description, optional image, and image alt text. Marker photos are not yet added.

### 3.4 Books (Resource Catalog)
* **Analysis:** Offers historical resources: *The Story of John Hendrix, Prophet of Oak Ridge* and the novel *Robertsville*.
* **Implementation:** Book cards with direct outbound Amazon purchase links, plus a list of local retailers (AMSE Gift Shop, K-25 History Museum, Museum of Appalachia, Hoskins Drug Store and Diner). Books and retailers are marked up with `schema.org/Book`, `schema.org/Offer`, and `schema.org/LocalBusiness` microdata to help local SEO discovery.

### 3.5 Geocaching
* **Analysis:** Connects the physical trail with the Geocaching network. Two caches cover the trail: GC5JYGG (markers 1–4) and GC5JYFR (markers 5–8).
* **Implementation:** A static page explaining geocaching, two cache cards (name, coordinates, difficulty and terrain ratings, and a direct link to the listing on Geocaching.com), and a note that a free Geocaching.com account is required to participate.
* **Current state:** Both caches are archived on Geocaching.com. A geocacher testimonials section is built but commented out until real quotes are collected. Re-activating the caches and restoring the testimonials is Phase 2. A live Geocaching API integration backed by Cloudflare Workers is a Phase 3 candidate.

### 3.6 Find Us (Location & Logistics)
* **Analysis:** Provides the trailhead location on Hendrix Drive across from Hampshire Court, park rules, and weather-dependent trail readiness guidance.
* **Implementation:** An address block with two actions: an *Open in Maps App* button (a native `geo:` link, coordinates `36.016966, -84.230019`) and a *Google Maps Directions* button. A trail-information list covers the ½-mile easy path, benches, and footwear and weather advice.
* **Map loading:** The Google Maps embed is click-to-mount. A placeholder panel ("Tap to load interactive map") renders first, and the iframe is only mounted after the visitor taps it, so the heavy map code never loads unless it is wanted. The embed uses Google's keyless embed URL rather than the Maps Embed API, so no API key or billing account is needed. The placeholder is a plain panel, not a static map image.

### 3.7 Donate to the Vision (Fundraising)
* **Analysis:** Drives the core charitable objective of building a physical Memorial Chapel to preserve pre-Oak Ridge family artifacts, photos, and histories.
* **Implementation:** A static page anchored by the Memorial Chapel vision and rendering image. Below it, an interactive tier selector (Trail Supporter, Creek Keeper, Chapel Builder, and a custom amount) builds the PayPal link for the chosen amount, and the *Donate with PayPal* button opens PayPal's hosted checkout in a new tab. A closing appeal repeats the volunteer and phone contact.
* **Stripe status:** Stripe is not yet wired in. There is no Stripe account, and the page contains no Stripe code. A placeholder Apple Pay domain-association file already sits in `public/.well-known/` so the domain-verification step is ready when Stripe is set up (see 4.1).
* **Progress meter:** A live campaign progress meter backed by Cloudflare D1 is a Phase 3 candidate once donation volume warrants the additional infrastructure.

---

## 4. Technology Stack Selection

| Layer | Technology | Selection Rationale |
| :--- | :--- | :--- |
| **Framework** | Astro 4 (static output) | Purpose-built for content-focused static sites. Astro ships zero JavaScript to the browser by default, and its HTML-like component authoring can be managed by non-developers over time, which matters for long-term volunteer maintainability. Static output needs no server adapter. |
| **Hosting Platform** | Cloudflare Pages | Git-integrated deployments, global CDN, automatic edge caching, and automated SSL. The free tier supports unlimited sites and requests. |
| **Styling Engine** | Tailwind CSS 3 (`tailwind.config.mjs`) | Utility-first compilation produces small stylesheets, which helps mobile users on the trail. Design tokens live in one config file (see section 6). |
| **Donation Gateway** | PayPal hosted checkout now; Stripe Payment Links planned (Square Payment Links is the fallback) | See 4.1. |
| **Fonts & Icons** | Google Fonts (Playfair Display, Source Serif 4, Inter) and Tabler icons, loaded from CDNs | See 4.2. |

### 4.1 Donation Gateway Decision
**PayPal at launch.** PayPal's hosted checkout is live today, requires no backend, and keeps card data off this site entirely.

**Stripe Payment Links next.** Stripe is the preferred addition because its hosted checkout surfaces Apple Pay and Google Pay automatically on supported devices, and Payment Links require zero backend code. Apple Pay needs a domain-verification file from Stripe placed in `public/.well-known/`. Because card data never touches the site's servers, the non-profit would qualify for SAQ A, the simplest annual PCI self-assessment. PayPal stays as an alternative giving option.

**Square is the fallback.** Square supports Apple Pay and Google Pay through its Web Payments SDK, but that path needs a Cloudflare Worker backend, which conflicts with the no-backend, volunteer-maintainable goal. Square Payment Links is a simpler no-backend middle ground. The final choice between Stripe and Square Payment Links has not been made.

### 4.2 Fonts and Icons via CDN
Google Fonts and the Tabler icon webfont are loaded from third-party CDNs through CSS `@import` in `global.css`. This keeps the repository small and the setup simple, but it has trade-offs:
* Rendering depends on the availability of two external hosts.
* `@import` is render-blocking, which works against the fast-on-the-trail goal.
* The Tabler URL currently references `@latest`, so an upstream release could change icons on the live site without any change to this repository.

Pinning the Tabler version and evaluating self-hosting are tracked in the README TODO.

---

## 5. Technical Performance & Mobile Optimization

### 5.1 Out-of-the-Box Edge Delivery
Compiled HTML, assets, and trail data are distributed directly to Cloudflare's global edge network. This removes origin computing delays, so a visitor loading the trail guide on-site in Oak Ridge is served from a nearby edge node.

### 5.2 Mobile-First Viewport Constraints
Because trail walkers will use the site outdoors, the design enforces several mobile-specific rules:
* **Thumb-Zone Action Bar:** On mobile breakpoints (< 768px), navigation collapses into a hamburger menu, and a fixed bottom bar anchors *Call*, *Find Us*, and *Donate* within immediate thumb reach.
* **Outdoor Readability:** Body copy is 18px (1.125rem) with generous line height on an off-white parchment background (`#F7F3EC`), with high-contrast primary text to reduce sunlight glare issues.
* **Tap Target Padding:** Interactive targets are a minimum of 48px × 48px (the mobile action bar is 56px tall).

### 5.3 Media Strategy
The legacy Weebly site's images are now hosted locally in `public/images/`. The design rules are:
* Images are standard compressed JPEGs added by dropping a file into `public/images/`. This is deliberately simpler than Astro's `<Image />` pipeline and WebP/AVIF conversion, so a volunteer can add a photo without build-tooling knowledge. Individual images should be kept modest in size.
* All trail-marker and historical images use `loading="lazy"`.
* All images carry explicit `width` and `height` (or a fixed-aspect wrapper) to prevent cumulative layout shift on low-bandwidth connections. A few images do not yet meet this rule; compressing existing images and adding the missing dimensions are tracked in the README TODO.

### 5.4 Frictionless Giving Integration
Donation flows are handled entirely by hosted checkout pages. When a donor taps the donate button, they are sent to PayPal, which handles payment. When Stripe Payment Links is added, Apple Pay (iOS) and Google Pay (Android) will initialize natively for single-tap checkout. No payment data passes through or is processed by this site, keeping PCI scope minimal.

### 5.5 Accessibility
WCAG 2.1 AA compliance is the standard for all pages:
* Conduct an accessibility audit using Lighthouse, WAVE, or Accessibility Checker.
* Provide descriptive `alt` text for all images, essential for screen readers and users with visual impairments.
* Keep navigation logical and consistent, with clear headings, concise link text, and ARIA attributes on interactive controls (the hamburger menu, for example, exposes `aria-expanded` and `aria-controls`).
* Enforce sufficient color contrast (see 6.4).
* Provide closed captions or a transcript for all video content, including the embedded YouTube documentary. Verifying captions on the video and deciding on a transcript are tracked in the README TODO.

---

## 6. Design System

The visual design draws on a historical, archival feel (aged paper, forest, creek water, copper) while staying readable outdoors on a phone. All tokens live in `tailwind.config.mjs`, and shared component classes live in `src/styles/global.css`. The README has a short practical guide to which class to use for what.

### 6.1 Palette

| Token | Hex | Role |
| :--- | :--- | :--- |
| `parchment` | `#F7F3EC` | Page background |
| `parchment-dark` | `#EDE6D8` | Alternating section background |
| `forest` | `#2D4A2D` | Header, hero sections, links, primary buttons |
| `forest-light` | `#3D6B3D` | Hover states, mobile menu background |
| `creek` | `#4A7C8E` | Secondary accent (water) |
| `creek-light` | `#6FA3B5` | Accent text on dark backgrounds, nav hover |
| `copper` | `#A0522D` | Eyebrow labels, dividers, donation call to action |
| `copper-light` | `#C47848` | Copper hover states |
| `ink` | `#1A1612` | Body text, footer background |
| `ink-soft` | `#3D3530` | Secondary body text |
| `stone` | `#8C8078` | Muted small text (captions, coordinates) |
| `stone-light` | `#BFB8AF` | Footer text, borders |

### 6.2 Typography and Icons
* **Display, `font-display`:** Playfair Display, a period-appropriate serif for headings.
* **Body, `font-body`:** Source Serif 4, readable with a historical feel.
* **UI, `font-ui`:** Inter, clean and modern for navigation, labels, and buttons.
* **Type scale:** `text-hero`, `text-section`, and `text-body` are responsive tokens defined in the Tailwind config.
* **Icons:** Tabler icons (webfont, `ti ti-*` classes).

### 6.3 Shared Components and Layout Rules
* **Buttons:** `btn-primary`, `btn-secondary`, and `btn-donate` (copper, reserved for giving).
* **Layout:** `section-container` (`max-w-4xl`, centered) and `section-pad` (vertical rhythm). Two-column layouts need more room, so `section-container` is widened selectively to `max-w-6xl` (as on the homepage) rather than changed globally.
* **Section pattern:** Pages open with a `bg-forest-textured` hero (with `hero-border` and an `eyebrow-light` label), followed by alternating `parchment` and `parchment-dark` sections.
* **Accents:** `eyebrow` and `eyebrow-light` section labels, `copper-divider`, `pullquote`, and `tier-active` (the selected donation tier).

### 6.4 Contrast
WCAG AA (4.5:1 for normal text) is the standard. Primary text, headings, buttons, and links pass comfortably. Two small-text uses currently fall short and are tracked in the README TODO: `stone` on parchment (about 3.5:1, and about 3.1:1 on `parchment-dark`) and `eyebrow-light` (`creek-light` on `forest`, about 3.6:1).

---

## 7. Content Editing Principle

Volunteer maintainability is a first-class constraint. Changing the words, photos, links, or amounts on the site should not require understanding layout or styling code. Where content is list-shaped, it lives as plain data (a JSON file for trail markers, and simple arrays at the top of each page for books, retailers, donation tiers, and geocache cards). Prose-heavy content, such as the homepage timeline and the trail rules on Find Us, is written directly in page markup, where a volunteer edits only the visible text and copies an existing block to add an entry. Sections that are not ready are hidden by commenting them out rather than deleting them, so re-enabling is a one-step change. The README has the practical guide to where each kind of content lives.

Two known exceptions to editing in a single place are listed in the README: the phone number appears in several files, and the PayPal link is set in two places in `donate.astro`.

---

## 8. SEO & Sharing

* Every page sets a unique title and description through `Layout.astro`, along with Open Graph title, description, and image tags.
* The site ships a favicon (`/favicon.svg`) and a default social sharing image (`/images/og-default.jpg`). The layout already references both files; creating them is tracked in the README TODO.
* The site publishes a `robots.txt` and an XML sitemap so search engines can index the six pages once the custom domain is live. Both are tracked in the README TODO.
* Structured data: the Books page uses `schema.org` microdata (see 3.4).

---

## 9. Hosting, Environments & Deployment

| Item | Detail |
| :--- | :--- |
| **Source** | GitHub repository `galapagosfinch/johnhendrixmemorial` |
| **Hosting** | Cloudflare Pages project `johnhendrixmemorial`, served at `johnhendrixmemorial.pages.dev` |
| **Build** | `npm run build`, with output directory `dist` |
| **Local development** | `npm run dev`, served at `http://localhost:4321` |
| **Previews** | Cloudflare Pages branch-based preview deployments are available, so changes can be reviewed on a branch before merging to production |
| **Custom domain** | Target is `johnhendrixmemorial.com`. The domain's DNS has moved to Cloudflare, but the DNS zone currently lives in a different Cloudflare account than the Pages project. Resolving that so the domain can be attached to the Pages project is tracked in the README TODO. |

Deploy and preview how-tos are in the README.

---

## 10. Phased Roadmap

| Phase | Scope |
| :--- | :--- |
| **Phase 1: Launch** | All six static pages, Astro + Tailwind, Cloudflare Pages hosting, PayPal donation with amount pre-selection, click-to-load Google Maps and YouTube embeds, static Geocaching page, and custom-domain cutover. No backend required. |
| **Phase 2: Stripe / Apple Pay & Geocaching** | Add Stripe Payment Links (or Square Payment Links) with Apple Pay and Google Pay, and complete the Apple Pay domain verification. Re-activate the archived geocaches and restore real geocacher testimonials. Still no backend required. |
| **Phase 3: Backend Enrichment** | Geocaching API integration (Cloudflare Workers) and a live donation campaign progress meter (Cloudflare D1). Triggered by audience growth and fundraising volume. |

Detailed task steps for each phase are maintained in the README.

---

## 11. Decisions & Rationale

* **Astro over Next.js:** The site is almost entirely static content, so shipping no JavaScript by default and HTML-like authoring beat a full-stack framework, and it keeps the codebase approachable for volunteers.
* **Static-only through Phase 2:** Every dynamic need (payments, maps, video) is handled by hosted third-party services, so there is no server to run, secure, or pay for until Phase 3.
* **PayPal first, Stripe next:** PayPal was already in use and works at launch with no new account. Stripe adds Apple Pay and Google Pay but needs an account and domain verification, so it follows launch. Square is kept as a fallback (see 4.1).
* **Hosted payment links over custom Workers:** Hosted checkout keeps card data off the site, limits PCI scope to SAQ A, and needs no backend code for volunteers to maintain.
* **Click-to-load video and map:** Both third-party embeds are heavy, and most visitors will not use them, so each loads only when tapped.
* **Keyless Google Maps embed:** It avoids an API key and billing account for a small nonprofit site.
* **Plain JPEGs in `public/images/`:** Adding a photo is a drag-and-drop for a volunteer, with no build-time image pipeline to learn (see 5.3).
* **Hide by commenting out, not deleting:** Unfinished or paused sections (geocacher testimonials, for example) can be re-enabled by uncommenting.
* **Static output without a Cloudflare adapter:** A static build needs none, which also avoids the adapter interfering with local development.
* **Re-sequenced roadmap:** Stripe and geocache reactivation moved to Phase 2 because neither needs a backend, leaving the Worker and D1 work for Phase 3.