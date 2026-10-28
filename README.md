# Warung Senja — QR Menu & WhatsApp Ordering Boilerplate

A production-minded, config-driven QR menu for restaurants in Malaysia. Guests scan a table QR code, browse the menu on their phone, build a cart, and send the order to the restaurant's WhatsApp with one tap.

Built for **Next.js 16 (App Router)**, React 19, TypeScript, Tailwind CSS 4 and Zustand 5.

## What is included

- Mobile-first menu for every table: `/table-1` … `/table-12`
- Table-aware cart with quantity controls and localStorage persistence
- WhatsApp checkout with table, line items, charges, totals and kitchen notes
- Config-driven branding, theme colours, tables, categories, dishes, prices and images
- Service charge + SST calculation, displayed transparently in the cart
- Accessible bottom-sheet cart with focus trap, Escape-to-close and safe-area padding
- Static table routes generated from the restaurant config
- Demo photography with attribution in [`IMAGE_CREDITS.md`](./IMAGE_CREDITS.md)

## Quick start

```bash
bun install
bun run dev
```

Then open [http://localhost:3000/table-1](http://localhost:3000/table-1). The root URL redirects to the default table.

`npm`, `pnpm` and `yarn` also work:

```bash
npm install
npm run dev
```

No database, API keys or environment variables are required.

## Customise the restaurant

Everything business-specific lives in [`src/config/restaurant.ts`](./src/config/restaurant.ts).

| What | Where |
| --- | --- |
| Name, monogram, tagline, cuisine, address, hours | top of `restaurant` |
| WhatsApp number | `whatsappNumber` — international format, digits only |
| Currency | `currency` (default `MYR`, locale `en-MY`) |
| Service charge and tax | `pricing` |
| Colours | `theme` — injected as CSS variables in `src/app/layout.tsx` |
| Tables | `tables`; `id` becomes the URL segment (`table-7` → `/table-7`) |
| Default table | `defaultTableId` |
| Menu categories | `categories` |
| Dishes | `dishes` — category, price, image, badge, spice level, availability |

### Add a table

1. Add an entry to `restaurant.tables`, for example `{ id: "table-13", label: "13", zone: "Terrace", seats: 4 }`.
2. Point the table's QR code at `https://your-domain.com/table-13`.
3. Keep `defaultTableId` pointing at an existing table.

### Add or change a dish

1. Put an image in `public/menu/` (roughly 1000–1200px on the long edge; 4:3 or square crops work best).
2. Add a dish object to `restaurant.dishes` with a unique `id`, `categoryId`, `price`, `image`, `description` and `isAvailable`.
3. Use `badge` for labels such as `"Chef's pick"` and `spiceLevel: 0–3` for the spice meter.

### Change the branding

Update the values in `restaurant.theme`. The layout exposes them as:

```
--ds-page  --ds-surface  --ds-ink  --ds-muted
--ds-brand --ds-brand-strong --ds-brand-foreground
--ds-accent --ds-accent-soft --ds-gold --ds-line
```

Components use semantic Tailwind tokens (`bg-brand`, `text-muted`, `border-line`, etc.), so no component edits are needed for a recolour. If you change the default table or the slug, the cart storage key changes with the slug, which is intentional.

### WhatsApp ordering

The demo number is a fictional `60123456789`. Replace it with the restaurant's WhatsApp Business number before going live. The order link uses `https://wa.me/<number>?text=<encoded message>`; the message is built in [`src/lib/order.ts`](./src/lib/order.ts) and includes:

- restaurant name and table number/zone
- quantities, dish names and line totals
- subtotal, service charge, SST and total
- an optional note for the kitchen

The guest reviews and sends the message in WhatsApp; this boilerplate intentionally has no server-side order database.

### Charges

`src/lib/pricing.ts` calculates:

```
subtotal      = sum(dish price × quantity)
service charge = subtotal × serviceChargeRate
tax            = (subtotal + service charge) × taxRate
total          = subtotal + service charge + tax
```

All money is rounded to two decimal places. Set either rate to `0` to hide that line from the cart.

## Project structure

```text
src/
├── app/
│   ├── [tableId]/page.tsx     # Server-rendered menu for one table
│   ├── globals.css            # Tailwind + design tokens + motion
│   ├── layout.tsx             # Fonts, theme vars, metadata, viewport
│   ├── page.tsx               # "/" → default table
│   ├── not-found.tsx          # Invalid table QR fallback
│   └── icon.svg
├── components/
│   ├── CartDrawer.tsx         # Floating cart bar + checkout sheet
│   ├── CartSync.tsx           # Rehydrate cart, bind it to the URL table
│   ├── DishCard.tsx           # Dish, price, spice meter, add/stepper
│   ├── Icons.tsx
│   ├── MenuGrid.tsx           # Category tabs + filtered dish grid
│   ├── MenuHeader.tsx         # Restaurant cover and table badge
│   ├── QuantityStepper.tsx
│   ├── RestaurantFooter.tsx
│   └── TableBadge.tsx
├── config/restaurant.ts       # Single source of truth
├── lib/
│   ├── format.ts              # Money/rate formatting
│   ├── order.ts               # WhatsApp message builder
│   └── pricing.ts             # Cart lines and totals
└── store/useCartStore.ts      # Zustand cart with persistence
```

## Cart behaviour

- Cart state is kept in Zustand and persisted to `localStorage` under `<slug>:cart:v1`.
- `persist` uses `skipHydration` so server HTML and the first client render match.
- `CartSync` rehydrates after mount and then pins the cart to the table in the URL.
- If a guest opens a different table, the previous table's cart is cleared to prevent food being sent to the wrong seat.
- Unknown dish ids in storage are ignored when lines and totals are calculated.

## Commands

| Command | Purpose |
| --- | --- |
| `bun run dev` | Start the dev server |
| `bun run build` | Production build (includes type checking) |
| `bun run start` | Serve the production build |
| `bun run lint` | ESLint |

## Deployment

Deploy the app to any Node-compatible host (Vercel, a container, or a Node server). There is no backend or database to provision. Set the restaurant details and WhatsApp number in the config before building; table pages are generated at build time.

For a QR code, encode the full table URL, for example `https://menu.example.com/table-4`. Rotate or reprint codes if table ids change.

## Demo content

The restaurant, menu, phone number and images are fictional demo content. Replace them before production. Dish photographs are licensed separately from the code; see [`IMAGE_CREDITS.md`](./IMAGE_CREDITS.md). If you use the demo images publicly, keep the attribution and comply with each licence.

## Notes for contributors and AI agents

- [`agents.md`](./agents.md) contains the implementation rules for future iterations.
- [`AGENTS.md`](./AGENTS.md) contains the framework-managed Next.js instructions plus project context.
- Next.js 16 in this repo has Cache Components and Partial Prefetching enabled. Read the bundled docs in `node_modules/next/dist/docs/` before changing routing or caching.
