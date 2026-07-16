# CLAUDE.md — Cerablus Coffee

Context for Claude Code. Read this before making changes.

## What we're building
A website for **Cerablus Coffee**, a specialty café in Nablus, West Bank. Two pages:
1. **Landing** — brand hero, quick highlights, contact/hours, links into the menu.
2. **Menu** — the real product: browse ~113 items, add to a cart, send the order to the café over WhatsApp.

This is a client project. The client provides menu content and photos; we build the site.

## Scope — build exactly this, nothing more
- Arabic interface, **RTL**. Some English is used as accent/label text (bilingual, modern).
- Menu with clear **categories**, **live in-menu search**, item **images + prices**.
- **Multi-pricing** per item: size (صغير/كبير) or portion (شخص/شخصين).
- **Add to cart** → **temporary in-memory cart** (no persistence needed) → **send order via WhatsApp**.
- **No** online payment, **no** accounts/login, **no** mobile app.
- Menu is **sheet-driven / updatable**: content lives in a data file (later fed from a published Google Sheet), never hardcoded in markup.

## Tech constraints
- **Plain static HTML/CSS/JS. No build step, no framework, no bundler.** Must open directly in a browser and deploy to Vercel as-is.
- **Mobile-first** — most visitors arrive from a WhatsApp link on a phone.
- Portable: everything should later drop cleanly into a framework if needed.
- Vanilla JS only. No jQuery. Google Fonts via `<link>` is fine.

## Brand
- **Colors** (define once in `:root`, use everywhere):
  - `--pine: #0c3d26` (primary dark green)
  - `--green: #006639` (secondary green)
  - `--gold: #cda05f` (accent — use sparingly)
  - `--mint: #e9f4ee` (light background tint)
  - Plus a warm off-white page background (e.g. `#f7f5f0`) and a neutral ink for text.
  - Reserve WhatsApp green (`#25D366`) **only** for order actions.
- **Logo**: `assets/cerablus-mark.svg` (icon) and `assets/cerablus-logo.svg` (full lockup). Both use `fill="currentColor"` so they recolor via CSS `color`. Icon in header/footer, full lockup in the hero.
- **Fonts**: a modern grotesk (Space Grotesk or Manrope) for Latin/display; **IBM Plex Sans Arabic** for Arabic body.

## Design language
Modern, minimal, confident — in the spirit of current Awwwards minimal winners:
- Oversized typography carries the page; strong hierarchy; big scale jumps.
- Generous whitespace, thin 1px hairline dividers, subtle asymmetry.
- Brand colors used **sparingly** against light/neutral backgrounds.
- Motion is subtle and meaningful: gentle hover shifts, scroll-reveal via `IntersectionObserver`.
- **Avoid** heritage/"eastern" ornamentation. Keep it clean and contemporary.
- The current front-runner directions are **F (type-led)**, **G (editorial)**, and **H (soft/product)** — match whichever the client picks.

## Data model
Menu content lives in `data/menu.js` as `window.MENU`. Schema mirrors the client intake sheet:

```js
window.MENU = {
  currency: "₪",
  categories: [ { id: "hot", name: "مشروبات ساخنة" }, ... ],
  items: [
    // single price:
    { id: "arabic-coffee", cat: "hot", name: "قهوة عربية", desc: "...", price: 8, image: "", available: true, featured: true },
    // multi-price (size or portion):
    { id: "cappuccino", cat: "hot", name: "كابتشينو", desc: "...",
      variants: [ { label: "صغير", price: 10 }, { label: "كبير", price: 13 } ],
      image: "", available: true }
  ]
};
```
Rules: an item has **either** `price` **or** `variants`. `image` empty → render a graceful placeholder tile. `available: false` → show "غير متوفر" and disable adding.

## WhatsApp order
- Phone number lives in **one** config constant (`CONFIG.PHONE`, digits only, e.g. `970590000000` → replace before launch).
- Cart line key = `itemId + "::" + variantLabel`.
- Order button builds a pre-filled Arabic message and opens `https://wa.me/<PHONE>?text=<encoded>`:
  ```
  مرحبا 👋 حابب أعمل هذا الطلب:

  • كابتشينو (كبير) ×2 — 26 ₪
  • كنافة ×1 — 18 ₪

  المجموع: 44 ₪

  الاسم:
  العنوان:
  ```
- With JS disabled, order links must still open a chat (hardcode a `wa.me/<PHONE>` fallback `href`, then upgrade it in JS).

## Suggested file structure
```
/index.html          landing
/menu.html           menu app
/styles.css          shared — all tokens in one :root block
/app.js              render + search + cart + WhatsApp
/data/menu.js        window.MENU (the only file that changes with content)
/assets/             logo SVGs, later item photos
```

## Conventions
- **Never** apply `letter-spacing` or `text-transform: uppercase` to Arabic text — it breaks letter joining.
- All colors/spacing/radius as CSS variables in one `:root` block. No hardcoded hex scattered in rules.
- Respect `prefers-reduced-motion`.
- Keyboard-accessible: real `<button>`/`<a>`, visible `:focus-visible`, `aria-*` on the cart drawer and controls.
- Keep it commented and readable. Prefer clarity over cleverness.
- **No `localStorage`/`sessionStorage`** — the cart is intentionally in-memory only.

## Do NOT
- Add a backend, database, accounts, or payment.
- Introduce a build step, framework, or npm dependencies.
- Hardcode menu items into HTML — they belong in `data/menu.js`.
- Add stock photos as placeholders; leave the placeholder tiles until the client's real photos arrive.
- Change brand colors or the logo.
