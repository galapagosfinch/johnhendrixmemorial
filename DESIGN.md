# Technical Design Document: Website Rebuild on Cloudflare Pages

**Project:** John Hendrix Memorial Prayer Walk Digital Modernization  
**Architect:** Steve Finch

---

This website represents the John Hendrix Memorial Prayer Walk in Oak Ridge, TN. The website will be rebuilt from scratch using a modern web stack with a focus on performance, accessibility, and user experience. It will leverage Cloudflare Pages for hosting. The majority of the site is static pages with text, images, and videos. There is a Donation page that uses Stripe Payment Links and PayPal to allow visitors to donate to the 501(c)(3) charity.

---

## 1. Architecture Overview

To transition the legacy Weebly/iPage site for the John Hendrix Memorial Prayer Walk into a modern, high-performance, and mobile-friendly web application, the new platform will be built entirely on **Cloudflare Pages**.

Given the primarily static nature of the historical content, a **Jamstack (JavaScript, API, and Markup)** architecture is the optimal approach. This design eliminates heavy server-side rendering, shifts delivery workloads to Cloudflare's global edge network, and ensures sub-second page loads — even on spotty mobile data connections out on the physical nature trail.

No serverless backend (Cloudflare Workers) is required for Phase 1. All dynamic functionality is delegated to hosted third-party services (Stripe, PayPal, Google Maps), keeping the architecture simple, maintainable by volunteers, and free of ongoing infrastructure overhead.

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
                  |  - Astro (SSG)        |
                  |  - Tailwind CSS       |
                  +-----------------------+
                               |
                               v
                +------------------------------+
                |       Third-Party Services   |
                +------------------------------+
                | - Stripe Payment Links       |
                | - PayPal Hosted Checkout     |
                | - Google Maps Embed API      |
                | - Geocaching.com (links)     |
                +------------------------------+
```

---

## 3. Structural Breakdown by Layout & Page Modules

Based on the analysis of the 6 core site paths, the application structure is decomposed into clean, componentized frontend views built with Astro components.

### 3.1 Global Layout Component (Navigation & Sticky Elements)
* **Analysis:** All existing pages utilize an identical header navigation layout.
* **Architectural Target:** A reusable Astro layout component using a responsive mobile-first hamburger menu.
* **Edge Optimization:** A persistent global footer showcasing the 501(c)(3) tax-exempt status, anchoring a high-contrast sticky viewport button targeting the donation funnel.

### 3.2 Meet John Hendrix (Homepage)
* **Analysis:** Focuses heavily on the historical narrative of John Hendrix ("The Prophet of Oak Ridge"), his 40 nights in the woods, and his Manhattan Project prophecies.
* **Architectural Target:** A structured, Markdown-driven narrative page. Media components (such as the embedded YouTube documentary video) will be lazy-loaded using native browser Intersection Observer APIs to avoid blocking the critical rendering path.

### 3.3 John Hendrix Memorial Prayer Walk (The Trail Guide)
* **Analysis:** Details a nature trail running along Hendrix Creek featuring eight descriptive story markers, local flora, train tracks, and a boxwood gravesite.
* **Architectural Target:** The 8 historical trail markers will be modeled inside a static JSON data file. On mobile viewports, the frontend maps this data array to a client-side accordion to minimize visual noise and enable interactive tracking for on-site hikers.

### 3.4 Books (Resource Catalog)
* **Analysis:** Offers historical resources including the novel *Robertsville* and a biography of John Hendrix for purchase.
* **Architectural Target:** Clean book card components with direct outbound Amazon purchase links. Local retailer listings (AMSE Gift Shop, K-25 History Museum, Museum of Appalachia, Hoskins Drug Store) rendered with `schema.org/Book` and `schema.org/LocalBusiness` microdata tags to enhance local SEO discovery.

### 3.5 Geocaching
* **Analysis:** Connects the physical trail location with the Geocaching network, featuring visitor testimonials and coordinates for the trail cache.
* **Architectural Target:** A static page with a clear description of the geocaching opportunity, the trail coordinates, and a prominent direct link to the cache listing on Geocaching.com. Users are advised that a free Geocaching.com account is required to participate. No API integration is needed in Phase 1. A dynamic Geocaching API integration backed by Cloudflare Workers is a candidate Phase 2 feature once the site has an established audience.

### 3.6 Find Us (Location & Logistics)
* **Analysis:** Provides critical logistical data regarding the trailhead location on Hendrix Drive across from Hampshire Court, park safety regulations, and weather-dependent trail readiness rules.
* **Architectural Target:** A performance-optimized Google Maps integration. Instead of loading the heavy JavaScript iframe embed on initial page load, a lightweight static map preview image renders first. Tapping it fires a native universal geo-link (`geo:36.01241,-84.233869`) to launch the device's native mapping app, or mounts the live Google Maps embed on click for desktop users.

### 3.7 Donate to the Vision (Fundraising)
* **Analysis:** Drives the core charitable objective of building a physical Memorial Chapel to preserve pre-Oak Ridge family artifacts, photos, and histories.
* **Architectural Target:** A compelling static page anchored by the Memorial Chapel vision and rendering image. Donation is handled by a Stripe Payment Link and the existing PayPal link — both are externally hosted, requiring zero backend code on this site. Apple Pay and Google Pay surface automatically on supported devices via Stripe's hosted checkout. A live campaign progress meter backed by Cloudflare D1 is a candidate Phase 2 feature once donation volume warrants the additional infrastructure.

---

## 4. Technology Stack Selection

To execute this architecture efficiently within Cloudflare's ecosystem, the following technologies have been selected:

| Layer | Technology | Selection Rationale |
| :--- | :--- | :--- |
| **Framework** | Astro (Static Site Generator) | Purpose-built for content-focused static sites. Astro ships zero JavaScript to the browser by default, resulting in near-perfect Lighthouse scores out of the box. Its component model allows for a clean, maintainable page structure without requiring full-stack JavaScript expertise — an important consideration for long-term volunteer maintainability. Astro has native Cloudflare Pages support with an official adapter, built-in image optimization (WebP/AVIF conversion), and straightforward HTML-like authoring that non-developers can manage over time. |
| **Hosting Platform** | Cloudflare Pages | Native git-integrated deployment pipelines, global CDN distribution, automatic edge caching, and automated SSL termination. Free tier supports unlimited sites and requests. |
| **Styling Engine** | Tailwind CSS | Utility-first compilation resulting in minimal, atomic production stylesheets, entirely avoiding heavy style payloads for mobile network users on the trail. |
| **Donation Gateway** | Stripe Payment Links + PayPal | Stripe Payment Links provide a Stripe-hosted, PCI-compliant checkout page requiring zero backend code. Apple Pay and Google Pay surface automatically on supported devices. A domain verification file placed in the Astro `public/` folder satisfies Apple Pay's domain registration requirement. The existing PayPal link is retained as an alternative giving option. The non-profit qualifies for SAQ A — the simplest annual PCI self-assessment — because card data never touches the site's own servers. |

---

## 5. Technical Performance & Mobile Optimization

### 5.1 Out-of-the-Box Edge Delivery
By leveraging Cloudflare Pages, compiled HTML, assets, and localized trail data are distributed directly to Cloudflare's global edge network. This removes backend origin computing delays, allowing a user loading the trail map on-site in Oak Ridge to receive data from the nearest physical server node instantly.

### 5.2 Mobile-First Viewport Constraints
Because trail walkers will interact with the site outdoors, the technical design enforces several mobile-specific constraints:
* **Thumb-Zone Sticky Footer:** On mobile breakpoints (< 768px), navigation collapses into a hamburger menu, and a persistent action bar anchors to the viewport bottom, putting *Call*, *Maps*, and *Donate Now* within immediate thumb reach.
* **Outdoor Readability:** Body copy is locked to a minimum of 16px with a text contrast ratio exceeding WCAG AA requirements on an off-white background (#FDFBF7) to minimize sunlight glare issues.
* **Tap Target Padding:** Interactive targets are set to a minimum of 48px × 48px to eliminate misclicks while users navigate the physical trail.

### 5.3 Media Strategy
The legacy site contains multiple historic images and assets. The new technical design mandates:
* Conversion of all image assets into modern, compressed `.webp` or `.avif` formats using Astro's built-in `<Image />` component.
* Enforcement of `loading="lazy"` attributes across all trail-marker and historical images.
* Explicit width and height dimensions on all image components to prevent cumulative layout shift (CLS) on low-bandwidth mobile connections.

### 5.4 Frictionless Giving Integration
Donation flows are handled entirely by Stripe's and PayPal's hosted checkout pages. When a donor taps a giving button, they are redirected to Stripe Payment Links where Apple Pay (iOS) or Google Pay (Android) initialize natively, enabling single-tap biometric checkout without requiring card entry on a mobile device. No payment data passes through or is processed by this site, eliminating PCI scope complexity entirely.

### 5.5 Accessibility
To maximize engagement from the largest number of visitors, WCAG 2.1 AA compliance will be pursued on all pages:
* Conduct a thorough accessibility audit using Lighthouse, WAVE, or Accessibility Checker to identify areas for improvement.
* Provide descriptive `alt` text for all images, essential for screen readers and users with visual impairments.
* Implement logical, consistent navigation with clear headings, concise link text, and ARIA attributes throughout.
* Enforce sufficient color contrast between text and background using tools such as Snook's Color Contrast Checker.
* Provide closed captions or transcripts for all video content, including the embedded YouTube documentary.

---

## 6. Phased Roadmap

| Phase | Scope |
| :--- | :--- |
| **Phase 1 — Launch** | All 6 static pages, Astro + Tailwind, Cloudflare Pages hosting, Stripe Payment Links + PayPal donation, Google Maps embed, static Geocaching page. No backend required. |
| **Phase 2 — Enrich** | Live donation campaign progress meter (Cloudflare D1 + Pages Functions), Geocaching API integration (Cloudflare Workers), enhanced trail interactivity. Triggered by audience growth and fundraising volume. |
