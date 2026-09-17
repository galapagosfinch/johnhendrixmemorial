# John Hendrix Memorial Prayer Walk — Website

Built with [Astro](https://astro.build) and [Tailwind CSS](https://tailwindcss.com), deployed on [Cloudflare Pages](https://pages.cloudflare.com).

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
    └── global.css             ← Global styles and Tailwind config

public/
└── .well-known/
    └── apple-developer-merchantid-domain-association  ← Apple Pay domain file
```
## Content Updates

### ✅ Completed
- Full site scaffold (6 pages: index, prayerwalk, books, geocaching, find-us, donate)
- Global layout with sticky mobile nav bar
- Design system (parchment/forest/creek/copper/ink palette, Tabler icons, Google Fonts)
- Images hosted locally in `public/images/`
- Interactive donation tier selector with PayPal pre-fill
- Both geocache URLs added (GC5JYGG markers 1–4, GC5JYFR markers 5–8)
- Deployed to Cloudflare Pages 

---

### 🔧 Short-term TODO

### Pre-selected Donate buttons
Create new URLs at PayPal, one for each of the pre-selectable amounts.
Update donate.astro to use those URLs when the button is selected.

#### Custom domain
Point `johnhendrixmemorial.com` to Cloudflare Pages (Transfer initiated)

#### Visual review
Do a full review on mobile and desktop before launch.

---

### 🗺️ Phase 2

#### Geocaching
- Re-activate geocaches (currently archived: GC5JYGG, GC5JYFR)
- In `src/pages/geocaching.astro`, uncomment testimonials section and replace the placeholders with real quotes from geocachers.

#### Stripe/Apple Pay setup
1. Create a Stripe account and register your domain for Apple Pay
2. In `src/pages/donate.astro`, replace the `stripeUrl` placeholder with your Stripe Payment Link URL. **Stripe account not yet created.**
3. Download the domain verification file from Stripe
4. Replace the contents of `public/.well-known/apple-developer-merchantid-domain-association`

---

### 📊 Phase 3
- Geocaching API integration via Cloudflare Workers
- Donation progress meter (Cloudflare D1)

## Deploying to Cloudflare Pages

1. Push this repository to GitHub
2. Log in to Cloudflare Pages → Create a project → Connect to Git
3. Build command: `npm run build`
4. Build output directory: `dist`
5. Done — Cloudflare handles SSL, CDN, and deployment automatically.
