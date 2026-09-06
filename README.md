# Amazon Landing Page

A Claude Code skill that turns any Amazon product URL into a premium landing page.

It reads the real product page (title, price, images, feature bullets, verified reviews) and writes a single self-contained HTML file. Nothing is invented: if the product has no verified 4-5 star reviews, the reviews section is left out.

## Install

```bash
npx skills add JayGPTPro/amazon-landing-page -g
```

Then open a new Claude Code session and run:

```
/amazon-landing-page https://www.amazon.com/dp/YOUR-ASIN
```

## Requirements

Chrome MCP has to be connected. The skill reads the live Amazon page through it, so without Chrome MCP it stops and tells you.

## What you get

One `index.html` with Tailwind (no build step), plus a folder of downloaded product images:

- Fixed nav with a price CTA and a sticky mobile buy bar
- Hero with a clickable image gallery and trust badges
- Animated social proof counters
- Features built from the Amazon bullets, rewritten short
- What's in the box, image gallery, FAQ
- Email capture, ready for a free Google Form (two variables to fill in)
- Real verified reviews only, with a link back to Amazon

Colors and typography are chosen per brand and product category, not from a fixed template.

## The elevation pass

The skill does not hand over its first draft. After the page is built and correct, it runs a
second pass that raises it two levels: a staggered hero entrance, a signature device built
from the product's own hardware and reused at several sizes, grain and mesh gradients under
the flat colour, a bento feature grid, a filmstrip gallery with snap and a progress bar, a
dark section for the one feature a photo cannot show, and micro-interactions on everything
you can click. The draft is kept as `index-v1.html` so you can see the difference.

The reference build is `B08V4PTCMR`, the Ortizan X10 speaker in pink.

## License

MIT. Use it, change it, ship it.

Built by [Jay Margaliot](https://jaygptpro.com).

## What changed in v2 (September 2026)

The skill now writes a design brief before it builds: a direction chosen for the product
(Expedition, Atelier, Playground, Lab, Pantry, Boutique, Workshop), a type pairing with
character, a palette pulled from the product images, and one signature visual device. The
patterns come from the best pages it has produced and live in `references/design-playbook.md`.
It also handles two Amazon traps found in testing: converted foreign-currency prices when the
item cannot ship to your country, and 500px images from the thumbnail strip (it now pulls the
1500px set), and it runs a QA pass before handing the page over.
