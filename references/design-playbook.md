# Design playbook

Distilled from the pages that came out best (the Maelstrom hydration pack redesign, the
Wave Runner football, the Woodenhouse cutting board, the Ella and Eden jojoba oil) and
from the ones that came out flat (the early Inter-on-white pages that all looked alike).

The pattern behind every good one: a **direction chosen for the product**, a **type pairing
with character**, a **palette lifted from the product**, and **one signature device**.

## 1. Directions

Pick the one that fits the product. Blend two only if you can name the blend.

| Direction | Fits | Feel | Hero recipe |
|---|---|---|---|
| **Expedition** | outdoor, sport, tools, auto, camping | dark ink, one hot brand color, mono labels, contours | dark hero, spotlight behind the product, floating spec chips, spec marquee under it |
| **Atelier** | beauty, skincare, jewelry, candles, home fragrance | warm paper, serif display, generous whitespace, thin rules | light hero, product on a soft radial, italic accent phrase, small-caps eyebrow |
| **Playground** | kids, toys, pool, party, pets | bright paper, rounded display font, chunky pills, confetti dots | light hero with a big tinted blob behind the product, oversized rounded CTA |
| **Lab** | electronics, audio, gadgets, chargers | near-black, cool gray, one electric accent, tight grid | dark hero, product large, spec table as the hero's second column |
| **Pantry** | kitchen, food, drinkware, cutting boards | cream, deep green or walnut, serif display, wood or linen textures | light hero, product big and angled, ribbon badge, section bands in brand color |
| **Boutique** | apparel, bags, footwear, watches | off-white, black, one accent, editorial type, big photos | photo-led hero, headline overlapping the image edge, minimal chrome |
| **Workshop** | hardware, garage, garden, safety, DIY | concrete gray, safety orange or yellow, condensed display | poster-style hero, huge condensed headline, striped hazard accent line |

## 2. Type pairings

Display + body + mono. Always three roles. Load only the weights you use.

| Direction | Display | Body | Mono / labels |
|---|---|---|---|
| Expedition | Bricolage Grotesque 600 to 800 | Inter or DM Sans | JetBrains Mono |
| Atelier | Fraunces or Cormorant Garamond 500 to 700 | Jost or DM Sans | IBM Plex Mono |
| Playground | Baloo 2 or Fredoka 600 to 800 | Nunito | Space Mono |
| Lab | Space Grotesk or Sora 600 to 700 | Inter | JetBrains Mono |
| Pantry | Fraunces 500 to 900 (italic for accents) | DM Sans | IBM Plex Mono |
| Boutique | Instrument Serif or Playfair Display 400 to 600 | Manrope | Space Mono |
| Workshop | Oswald or Archivo 600 to 900 | Inter | Roboto Mono |

Rules: headline tracking tightens as size grows (`-0.01em` at 3rem, `-0.03em` at 5rem).
Body 17 to 19px on desktop. Mono labels 11px, uppercase, `letter-spacing:0.18em`.

## 3. Palette from the product

Five tokens, named after the product, registered in `tailwind.config`:

```js
colors: {
  trail: { DEFAULT:'#F2622A', light:'#FF8048', dark:'#D24E1B' },   // the product's own color
  hydro: { DEFAULT:'#2BB3D4', dark:'#1A8AA6' },                     // second color from the product
  ink:   { DEFAULT:'#13171B', 700:'#21272E' },
  bone:  { DEFAULT:'#F4EEE4', 100:'#FAF6EE', 200:'#E9DFCE' },      // paper, never pure white
}
```

How to pick: open the main image, name the two dominant product colors, use the stronger
one for CTAs and accents, the second for secondary chips. Paper is warm (`#F4EEE4`,
`#FAF6EE`) or cool (`#F3F5F7`) to match the product's temperature. Pure `#fff` sections
only as cards on the paper.

## 4. Signature devices (choose one)

**Topographic contours** (Expedition). SVG data URI of wavy paths, stroke in the brand
color at 10% on dark, ink at 4% on light. `background-size:900px auto`.

```css
.topo-dark{background-color:#13171B;background-image:url("data:image/svg+xml,%3Csvg width='800' height='600' viewBox='0 0 800 600' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='none' stroke='%23F2622A' stroke-opacity='0.10' stroke-width='1.2'%3E%3Cpath d='M-50 120 C 150 60 300 180 450 120 S 700 40 850 120'/%3E%3Cpath d='M-50 180 C 150 120 300 240 450 180 S 700 100 850 180'/%3E%3Cpath d='M-50 240 C 150 180 300 300 450 240 S 700 160 850 240'/%3E%3Cpath d='M-50 300 C 150 240 300 360 450 300 S 700 220 850 300'/%3E%3Cpath d='M-50 360 C 150 300 300 420 450 360 S 700 280 850 360'/%3E%3Cpath d='M-50 420 C 150 360 300 480 450 420 S 700 340 850 420'/%3E%3C/g%3E%3C/svg%3E");background-size:900px auto}
```

**Spotlight hero**. A radial glow in the brand color behind the product on a dark hero.
`radial-gradient(circle at 50% 42%, rgba(BRAND,.35) 0%, rgba(BRAND,.08) 32%, transparent 62%)`.

**Floating spec chips**. Two small dark pills absolutely positioned over the hero image,
each quoting one real spec ("2L Insulated Bladder", "8 Smart Pockets"), floating on a 6s
ease with different delays. Hidden under `sm`.

**Spec marquee**. A full-width band in the brand color under the hero, mono uppercase,
6 specs separated by slashes, scrolling 30s linear, duplicated once for the loop, paused
on hover. The single loop on the page; everything else plays once. This is the ONLY place
a row of specs is allowed. Never a stat strip of big counters (see "What not to do").

**Outlined numerals**. Big feature numbers with `-webkit-text-stroke` in the brand color
at 35% and transparent fill. Pairs with a bento grid.

**Bento feature grid**. One large tile (2 columns, dark, with the outlined numeral and the
strongest feature) plus five regular tiles. Corner icon in every tile.

**Editorial pull-quote** (Atelier, Boutique). One review, set huge in the display italic,
between two thin rules, author in mono below.

**Section bands** (Pantry, Playground). Alternate paper and a deep brand-color band
(the green of the packaging, the walnut of the board) with the copy in paper color.

## 4b. What not to do

- **No stat strip.** The row of four big numbers with mono labels ("60W / 8h / 4.8 stars /
  1,046 customers") is the signature of an AI-built page. Jay killed it on sight. Numbers
  go inside the sentence that earns them.
- No three equal feature cards with an emoji in each.
- No gradient text on white. No purple-on-white gradients at all.
- No "Trusted by X customers" headline unless X comes from the listing.
- No section that a competitor could paste onto their own page unchanged.
- **Never crop an infographic.** Amazon secondary images carry the seller's callouts at the
  edges; a negative margin or `object-cover` eats the labels. Infographics sit fully inside
  the grid. Only photographs may bleed or be cropped.
- **Full-bleed sections need a real photo.** Many "lifestyle" listing images have an
  infographic panel baked into one third of the frame. Crop that panel off with PIL into a
  `photo-N.jpg` before using the image edge to edge, and position the subject with
  `object-position`.

## 5. Copy rules

- Headline: 3 to 6 words, the promise. "Hydrate hands-free. Carry it all." not "20L
  Hydration Backpack with 2L Bladder".
- Subhead: two sentences, who it is for and the one thing it does best. From the bullets.
- Feature titles: 2 to 4 words. Body: one sentence.
- Section eyebrows: mono, uppercase, product-specific ("Built for the trail", "Up close",
  "Before you hit the trail"). Never "Features" alone when you can say more.
- Review headline uses the real count and the audience: "Trusted by 1,046 adventurers."
- Final CTA: two short lines, the second in the brand color.
- No em dashes, no en dashes. Ranges use "to".

## 6. Motion

- Reveal on scroll (fade up 30px, 0.8s, `cubic-bezier(.16,.8,.3,1)`), hero copy from the
  left and product from the right. Gated behind `html.js-anim` so nothing is hidden when
  JS fails. Honor `prefers-reduced-motion`.
- No counters. Numbers sit still inside sentences.
- CTA: soft pulse ring in the brand color, 2.6s.
- Product image: slow float, 6s, 14px.
- Nothing else loops. Motion is seasoning.

## 7. The Maelstrom skeleton (reference)

Order that worked: nav (pill, blurred), dark hero with spotlight and chips, spec marquee,
bento features, in-the-box beside an infographic image,
gallery, dark VIP band, reviews on cards with a big real count, FAQ, final band, footer,
sticky mobile bar. Each section 80 to 112px of vertical padding, alternating paper tones
so the eye never sees two identical backgrounds in a row.
