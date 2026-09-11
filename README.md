# tryoud

Marketing site for tryoud — custom websites plus photo, video, and drone content
for mid-sized German companies. Static Astro + Tailwind, deployed on Cloudflare Pages.

## Commands

| Command        | Action                                    |
| :------------- | :---------------------------------------- |
| `pnpm install` | Install dependencies                      |
| `pnpm dev`     | Dev server on http://localhost:4321       |
| `pnpm build`   | Build the static site to `./dist/`        |
| `pnpm preview` | Serve the production build locally        |

## Structure

```
content/site.json          German + English copy, contact details, form endpoint
src/pages/index.astro      Homepage; also holds the language-switch script
src/pages/{imprint,privacy,404}.astro
src/layouts/Layout.astro   <head>, meta tags, JSON-LD
src/components/Navigation.astro
src/components/sections/   One file per homepage section
src/styles/global.css      Design tokens, .section/.shell rhythm, .card, .btn
```

## How the two languages work

German is the rendered default: every translatable node carries `data-i18n="some.key"`
with the German text inline, so the HTML that ships (and that Google indexes) is German.
The script at the bottom of `index.astro` snapshots those nodes on load and swaps in
`content/site.json`'s `en` values when the visitor picks EN; switching back to DE restores
the snapshot. The choice is kept in `localStorage` and mirrored to `?lang=en`.

Two consequences worth knowing:

- **Defaults are keyed by node, not by i18n key.** A key can appear on more than one
  element (`services.content.title` is on both the service card and the contact
  `<select>`), and keying by string would let the last one overwrite the others.
- **New German copy goes in the component**, then the English counterpart goes in
  `site.json`. A key missing from `site.json` simply stays German.

## Prices

Starting prices live in three places and must be changed together:

- `src/components/sections/Services.astro` — the `from` field per service
- `src/components/sections/Faq.astro` + `content/site.json` — the "Was kostet eine Website?" answer
- `src/layouts/Layout.astro` — the `offers` array behind the `ProfessionalService` JSON-LD

Google flags structured data that disagrees with the visible page, so these three
must not drift apart.

## Scroll reveals

Sections fade in via `.reveal-on-scroll` + an IntersectionObserver. The hidden state is
scoped to `html.js` (set by an inline script in `Layout.astro`), so if the script never
runs the content is simply visible rather than an empty page.

## Screenshots

Hero and Work thumbnails come from WordPress's public mshots service, which renders each
client site on demand. A cold cache can take a few seconds and returns a blank frame in
the meantime, which is why the cards carry their own placeholder tone. Committing real
screenshots to `public/` would remove that third-party dependency.
