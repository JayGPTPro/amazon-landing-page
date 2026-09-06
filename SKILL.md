---
name: amazon-landing-page
description: Build a premium, brand-specific landing page from any Amazon product URL. Reads the real listing through Chrome (title, images, bullets, price, verified reviews), picks a bold design direction that fits the product, and writes one self-contained HTML file with a gallery, real reviews, FAQ, email capture and a sticky mobile buy bar. Use when the user says "build a landing page", "create a product page", "amazon landing page", or gives an Amazon product URL or ASIN.
argument-hint: "<amazon-product-url-or-ASIN>"
user-invocable: true
---

# Amazon Product Landing Page Builder

You are about to build a premium landing page for the Amazon product at **$ARGUMENTS**.
If the argument is a bare ASIN, the URL is `https://www.amazon.com/dp/<ASIN>`.

The bar is not "a clean template with the product dropped in". The bar is a page the brand
could have commissioned from a good agency: a design direction chosen for THIS product, a
distinctive type pairing, a palette pulled from the product itself, one signature visual
device, and copy that sounds like a person. Read `references/design-playbook.md` before
you write a line of HTML. It holds the patterns that made the best pages good.

Building it in one pass is not how the good ones happen. Phase 5 gets a correct page.
**Phase 6 is where it becomes worth shipping, and it is not optional.**

**Reference build: `B08V4PTCMR`, the Ortizan X10 speaker in pink.** That is the standard.
Its first draft was clean and forgettable. The elevation pass gave it a hero that arrives
in eight staggered beats, a signature light ring built from the product's own RGB feature
and reused at three sizes, grain and mesh under the colour, a bento feature grid, a
filmstrip gallery, and a near black section for the light show. Same data, same product,
same reviews. If your finished page would not sit next to that one, you stopped at Phase 5.

## Requirements

- **Chrome MCP is required.** Test it first. If it is not available, stop and tell the user
  to connect Chrome to Claude Code.
- One HTML file with Tailwind via CDN (no build step) plus an `images/` folder.
- Every fact on the page comes from the real Amazon page. Never invent a review, a number,
  a badge or a spec.
- No em dashes and no en dashes anywhere in the copy. Periods and commas.

## Process

### Phase 1: Setup

1. Create `amazon-landing-page-[short-product-name]/` in the current directory
   (lowercase, hyphens, the product not the ASIN: `amazon-landing-page-tongue-drum`).
2. Create `images/` inside it.

### Phase 2: Extract product data via Chrome MCP

1. Open the Amazon URL in Chrome MCP. Wait about 3 seconds for the page to settle.
2. Extract with JavaScript execution:

```javascript
// Product basics
({
  title: document.getElementById('productTitle')?.textContent?.trim(),
  price: document.querySelector('.a-price .a-offscreen')?.textContent?.trim(),
  listPrice: document.querySelector('.basisPrice .a-offscreen, .a-text-price .a-offscreen')?.textContent?.trim(),
  rating: document.querySelector('#acrPopover')?.title || document.querySelector('.a-icon-alt')?.textContent,
  reviewCount: document.querySelector('#acrCustomerReviewText')?.textContent?.trim(),
  brand: document.querySelector('#bylineInfo')?.textContent?.trim(),
  badges: document.querySelector('#acBadge_feature_div, #zeitgeistBadge_feature_div')?.textContent?.trim(),
  boughtRecently: document.querySelector('#social-proofing-faceout-title-tk_bought')?.textContent?.trim()
})
```

```javascript
// Feature bullets
Array.from(document.querySelectorAll('#feature-bullets .a-list-item'))
  .map(el => el.textContent?.trim()).filter(t => t && t.length > 10)
```

```javascript
// Product details table (materials, dimensions, weight, what is included)
Array.from(document.querySelectorAll('#productDetails_techSpec_section_1 tr, #detailBullets_feature_div li, #productOverview_feature_div tr'))
  .map(r => r.textContent.replace(/\s+/g,' ').trim()).filter(Boolean).slice(0,30)
```

```javascript
// High-res image URLs, gallery order
Array.from(document.querySelectorAll('#altImages .a-button-thumbnail img'))
  .map(img => img.src.replace(/\._.*_\./, '._SL1500_.'))
  .filter(s => s.includes('images/I/'))
```

3. Scroll to the reviews and extract REAL reviews only:

```javascript
Array.from(document.querySelectorAll('[data-hook="review"]')).map(r => ({
  stars: parseFloat(r.querySelector('[data-hook="review-star-rating"] .a-icon-alt')?.textContent || '0'),
  title: r.querySelector('[data-hook="review-title"] span:last-child')?.textContent?.trim(),
  body: r.querySelector('[data-hook="review-body"] span')?.textContent?.trim(),
  author: r.querySelector('.a-profile-name')?.textContent?.trim(),
  date: r.querySelector('[data-hook="review-date"]')?.textContent?.trim(),
  verified: !!r.querySelector('[data-hook="avp-badge"]')
})).filter(r => r.stars >= 4 && r.body && r.body.length > 20 && r.verified)
```

If `body` comes back empty (Amazon changes its review markup often), fall back to
`r.innerText` per review and split it yourself: the first line is the author, then the star
line, then the title, then "Reviewed in ... on <date>", then "Verified Purchase" or "Vine
Customer Review of Free Product", then the body. Vine reviews are real but not verified
purchases: you may use them, labeled "Vine reviewer", never as "Verified".

If fewer than 3 usable reviews come back, open `https://www.amazon.com/product-reviews/<ASIN>`
and run the same extraction there. Trim long reviews to the strongest 1 to 3 sentences, keep
the words as written.

**Price traps.** If the page shows a foreign currency (ILS, EUR) or "cannot be shipped to
your location", the buy box is hidden and `.a-price` is a converted reference price. Get the
real USD figures from the page's embedded data instead:

```javascript
var h=document.documentElement.outerHTML;
({usd:[...new Set(h.match(/\$\s?\d{2,4}\.\d{2}/g)||[])].slice(0,12),
  list:(h.match(/formattedFullPrice[^$]*(\$[\d,]+\.\d{2})/)||[])[1],
  save:(h.match(/percentageDisplayString[^%]*?(\d+)%/)||[])[1]})
```

The price to pay appears with `"currencyCode":"USD"` next to it; the list price as
`formattedFullPrice`; the saving as `percentageDisplayString`. Confirm the three agree
(price + saving = list) before using them.

**Image trap.** `#altImages` thumbnails are 40px sources and the `_SL1500_` swap on them
returns 500px files. The full-size set is in the page script under `colorImages`:

```javascript
[...document.scripts].map(s=>s.textContent).find(t=>t.includes('colorImages')).match(/"hiRes":"([^"]+)"/g).map(x=>x.split('"')[3])
```

That list is in gallery order and includes every image, 1500px. Video thumbnails
(`play-icon-overlay` in the URL) are not images; skip them.

### Phase 3: Download images

```bash
curl -sL "[image-url]" -o images/product-1.jpg
```

Name them in gallery order (`product-1.jpg` is the main image). Check every file is over
10 KB; a 9-byte file is a blocked download, drop it and move on. Never retry a blocked URL.

### Phase 4: Design direction (write it down before building)

Open the downloaded images with the Read tool. Look at the product, its packaging, the
lifestyle shots, and the brand's own colors. Then write a six-line brief into
`DESIGN.md` in the project folder:

1. **Audience and mood** in one sentence. Who buys this and how the page should feel.
2. **Direction**, one of the archetypes in the playbook (Expedition, Atelier, Playground,
   Lab, Pantry, Boutique, Workshop) or a named blend. Say why it fits this product.
3. **Type pairing**: display font + body font + mono for labels. Pick from the playbook
   table for the direction. Inter on its own is never the answer.
4. **Palette**: 5 tokens with hex codes, all pulled from the product images or packaging:
   `brand` (the product's own color), `accent` (contrast for CTAs), `ink`, `paper`, and
   one `tint`. Name them after the product ("trail", "hydro") so the code reads well.
5. **Signature device**: exactly one from the playbook (topographic contours, spec marquee,
   spotlight hero, floating spec chips, outlined numerals, bento feature grid, editorial
   pull-quote, product-color section bands). One. Two is a costume party.
6. **Headline**: three words to six, the promise not the product name. Write three
   options and choose one.

This brief is the contract for Phase 5. It is what stops every page from looking like the
previous one.

### Phase 5: Build the landing page

Create `index.html`. Sections in this order, each with an id:

1. **Nav**: fixed, pill or bar with backdrop blur, brand mark (initial in a colored square
   plus the name), section links (hidden on mobile), "Buy on Amazon · $price" button.
2. **Hero**: the direction's hero recipe from the playbook. Eyebrow row (badge as extracted,
   plus a mono category label), the headline in display type at 4.5 to 5rem with one
   accented phrase, a two-line subhead written from the bullets, stars with the real
   rating and count linking to Amazon reviews, price (with list price struck through if
   there is one), the CTA with a soft pulse, three trust chips (30-Day Returns, Ships via
   Amazon, and one that is true for this product). Image side: main image with a slow
   float, one or two floating spec chips that quote real specs, and a thumbnail row that
   swaps the main image.
3. **Proof line, not a stat strip.** No row of big counters ("60W / 8h / 4.8 stars"). Every AI
   landing page has one and it reads as filler. If the product has one number that sells it,
   put that number inside the feature that earns it, in a sentence. Rating and review count
   live in the hero and in the reviews section, nowhere else.
4. **Features** (`id="features"`): 5 to 6 features rewritten from the bullets, short
   title plus one sentence each. Not a grid of equal cards. Give the strongest feature its
   own full-width moment (a large image beside big type, or a bento tile that spans two
   columns), then the rest smaller. Where the listing has an infographic for a feature, use
   it as that feature's image. Icons are inline SVG, never an icon font.
5. **In the box / detail**: an image beside a checklist, from the bullets and the details
   table. Include dimensions and materials when the listing has them.
6. **Gallery** (`id="gallery"`): every product image, masonry or 3-column, hover zoom.
   Amazon infographic images go here as they are; they carry the seller's own callouts.
7. **Email capture**: dark or brand-colored band, one input plus button, VIP-list copy in
   the product's voice. Keep the two config constants at the top of the script:
   ```
   const GOOGLE_FORM_URL = ''; // Seller fills this in
   const EMAIL_FIELD_ID = '';  // Seller fills this in
   ```
   On submit with empty config, show the success state and a link to Amazon anyway.
8. **Reviews** (`id="reviews"`): 4 to 6 verified reviews, each with a bold title line,
   stars, author, date, "Verified" chip. Headline uses the real count ("Trusted by 1,046
   adventurers"). Link "Read all reviews on Amazon".
9. **FAQ** (`id="faq"`): 5 to 6 questions a real shopper asks about THIS product type,
   answered from the bullets and details. Accordion, one open at a time.
10. **Final CTA**: a full-width band in the brand color or ink, headline, price, button,
    two trust lines.
11. **Footer**: brand mark, section links, disclaimer: "Independent product page. [Brand]
    is a registered trademark of its owner. Purchase fulfilled by Amazon.com."
12. **Sticky mobile bar**: price plus Buy button, appears after the hero scrolls out.

Design ambition, before the technical rules. A page that passes QA can still be flat. Read
the finished layout against these and fix what fails:

- **Rhythm.** No two consecutive sections share a background tone or a layout shape. Dark,
  paper, full-bleed photo, paper, dark. Two columns, then one, then a grid, then a band.
- **One full-bleed photo section.** The best lifestyle image from the listing, edge to edge,
  with a short line of display type on it and nothing else. This is the breath in the page.
- **Scale contrast.** At least one headline at 5rem or more, and body copy that stays at 17
  to 19px. Big and small next to each other is what makes a page feel designed.
- **Asymmetry somewhere.** A headline that overlaps the edge of an image, a card that sits
  off the grid, a photo that bleeds off the right edge while the copy sits in the grid.
- **The product in the copy's color.** Pull one image out of its white background with a
  soft drop shadow or place it on a tinted plate, so the product sits IN the page instead
  of on top of it.
- **Whitespace as a material.** 96 to 128px between sections on desktop. Cramped is the
  first thing an eye reads as cheap.
- **Details that only a person would add.** A mono label with the ASIN or the pack's
  capacity in the corner of the hero. A hand-set pull quote. A tiny drawn icon that matches
  the product, not a generic one.

Technical rules:

- `<script src="https://cdn.tailwindcss.com">` with a `tailwind.config` that registers the
  palette tokens and font families. Google Fonts via `<link>` with preconnect.
- Custom CSS in one `<style>`: reveal classes gated behind a `js-anim` class on `<html>`
  so content is never stuck invisible, `prefers-reduced-motion` respected, the signature
  device, the CTA pulse, the accordion.
- All JavaScript at the bottom: gallery swap, scroll reveal, accordion, nav state on
  scroll, sticky bar, email form.
- Scroll reveal is a `requestAnimationFrame` throttled scroll pass, not an
  IntersectionObserver that unobserves on first hit. Elements are removed from the list once
  revealed, so nothing can be skipped by a fast scroll or an anchor jump:
  ```js
  var revealEls = Array.prototype.slice.call(document.querySelectorAll('.rv'));
  function revealPass(){
    for(var i = revealEls.length - 1; i >= 0; i--){
      if(revealEls[i].getBoundingClientRect().top < window.innerHeight - 40){
        revealEls[i].classList.add('in'); revealEls.splice(i, 1);
      }
    }
  }
  ```
  Call it once on load, then on `scroll` and `resize` behind a rAF tick.
- Every Amazon link: the product URL, `target="_blank" rel="noopener"`.
- Mobile first. Test the hero at 390px in your head: headline 2.6rem, image under the copy,
  thumbs scroll horizontally, chips hidden.
- Alt text on every image, from the listing.

### Phase 6: The elevation pass (do not skip this)

**The page you just built is the draft, not the deliverable.** Phase 5 gets the structure,
the real data and a defensible direction onto the screen. It reliably produces a page that
is correct and slightly flat. The difference between "clean" and "a brand paid for this" is
made here, and it is made on purpose, not by luck.

Copy the draft to `index-v1.html` first, so the lift is visible and reversible. Then go
back through the page and raise it two levels. Not a restyle. Each lane below changes what
the page *does*, and each one has a concrete implementation.

1. **Give the hero an entrance.** A page that fades in as one block reads as a template.
   Stagger it: put a `.hi` class on the eyebrow, each line of the headline, the subhead, the
   stars, the price and the CTA, each with its own `style="--d:.12s"` delay. Gate the whole
   thing on a `loaded` class added by a double `requestAnimationFrame`, never on
   `window.onload`, which would hold the hero hostage to the last image.

   ```css
   html.js-anim .hi{ opacity:0; transform:translateY(22px); }
   html.js-anim.loaded .hi{ animation:rise .9s cubic-bezier(.16,.8,.3,1) forwards; animation-delay:var(--d,0s); }
   @keyframes rise{ to{ opacity:1; transform:none; } }
   ```
   ```js
   requestAnimationFrame(function(){ requestAnimationFrame(function(){
     document.documentElement.classList.add('loaded'); }); });
   ```

2. **Build the signature device for real.** In Phase 5 it is usually a gradient and a
   promise. Now make it out of the product itself. The Ortizan speaker has an RGB light
   ring, so the page got an actual ring: a `conic-gradient` through the product's own light
   colours, an animatable angle via `@property --a`, a blurred masked halo behind the
   product, and the same ring reused at 9px as the nav mark and as a status dot. One device,
   built once, reused three times at three sizes. That repetition is what reads as identity.

   ```css
   @property --a { syntax:'<angle>'; inherits:false; initial-value:0deg; }
   .ring{ --a:0deg; background:conic-gradient(from var(--a), var(--hot), var(--sun), var(--cyan), var(--hot)); }
   .halo{ background:conic-gradient(from 0deg, var(--hot), var(--sun), var(--cyan), var(--hot));
          mask:radial-gradient(farthest-side, transparent 46%, #000 63%, #000 80%, transparent 100%);
          filter:blur(26px); animation:spin 18s linear infinite; }
   ```

3. **Put material under the colour.** Flat hex fills are the tell. Add, in this order:
   a mesh gradient ground (three or four soft `radial-gradient` blobs in the palette over
   the paper colour), a grain overlay (inline SVG `feTurbulence` at `opacity:.09`,
   `mix-blend-mode:soft-light`) on the hero and the dark sections, and `backdrop-filter`
   glass on the nav and any floating chips. Cheap, and it is most of the perceived jump.

4. **Break the equal grid.** If the features are still six cards of the same size, convert
   to a bento: one tile spanning two columns with the strongest claim and a real image, one
   tall tile, four normal. Give tiles a lift on hover
   (`transform:translateY(-5px)` plus a deeper shadow) and scale the image inside to 1.04.

5. **Turn the gallery into a filmstrip.** Replace the masonry with a horizontal
   `scroll-snap-type:x mandatory` strip, numbered `lbl` captions under each frame, prev and
   next buttons, and a progress bar that tracks `scrollLeft`. It bleeds off the right edge,
   which is the asymmetry the page was missing.

6. **Give the best feature its own room.** Find the one thing the product does that a
   photograph cannot show and build a section for it, in the opposite tone to the rest of
   the page. On the Ortizan that is a near black "night" section for the light show, sitting
   between two bright ones. This is the section a competitor cannot paste onto their page.

7. **Micro-interactions everywhere a finger goes.** Buttons lift 2px on hover and settle to
   `scale(.98)` on press, with a blurred glow ring behind the primary CTA. Swap the FAQ from
   `max-height` to `grid-template-rows:0fr` to `1fr`, which animates to the true height with
   no magic number and no clipped answers. Thumbnails lift on hover.

8. **Let the type get loud.** One display face with real character, one number face for the
   price and the spec numerals, tracking tightened to `-.035em` at the top size. Reserve
   gradient text for exactly one phrase in the headline. Body stays 17 to 19px.

Housekeeping while you are in there: hoist any icon you repeat more than twice into an
`<svg><symbol>` sprite at the top of the body and call it with `<use href="#star"/>`. A
five pointed star pasted thirty times is thirty times the bytes and the first thing that
looks generated.

**The bar.** Open `index-v1.html` and `index.html` side by side. If you cannot name three
things that changed *structurally*, you restyled instead of elevating. Go again.

### Phase 7: QA before you show it

1. Open `index.html` in the browser (Chrome MCP or `open index.html`).
2. Take a full-page screenshot and LOOK at it. Then check, in this order:
   - Every image loaded (no broken icons, no 9-byte files).
   - No text overflows its box at desktop and at 390px width.
   - The headline is the promise, not the Amazon title.
   - Nothing on the page is invented: every number, badge and review traces to Phase 2.
   - No em or en dash anywhere: `grep -c "—\|–" index.html` must print 0.
   - The reveal classes do not hide content when JS runs late (scroll to the bottom, all
     sections visible).
   - No infographic is cropped by a bleed or a negative margin; every callout at its edges
     is readable. Full-bleed photos show no baked-in text panel.
   - The page does not look like the last page you built. If it does, the direction in
     DESIGN.md was not followed; fix the page, not the brief.
   - Phase 6 actually happened. `index-v1.html` exists and the two files differ by more
     than colours.
3. Fix what you found. Do not report "done" with a known defect.

**Judge animation only in a visible window.** A hidden or backgrounded browser pane freezes
`requestAnimationFrame` and every CSS animation, so all your reveal and entrance elements
measure at `opacity:0` and the page looks catastrophically broken when nothing is wrong.
Before you believe that reading, check `document.visibilityState`. If it says `hidden`,
front the tab and measure again rather than "fixing" working code.

### Phase 8: Hand over

Tell the user, in four lines: the design direction you chose and why, what the page
contains (sections, number of real reviews used), the two things they must do before
publishing (put their attribution link on the buy buttons, paste the Google Form URL and
field id for email capture), and the path. Then open the page.

## Important rules

- NEVER fabricate reviews, ratings, counts, badges or specs. If a section has no real data,
  leave it out and say so.
- The headline is a rewritten promise, never the Amazon title.
- One design direction per page, written down first, followed to the end.
- No Inter-only typography, no purple-gradient-on-white, no generic dark theme by default.
- FAQ answers come from the listing, phrased plainly. Unknown is "check the listing", not
  a guess.
- The page should look like the brand made it, not like a template with a logo swapped.
- Never hand over the Phase 5 draft. The elevation pass in Phase 6 is part of the job, not
  an upsell, and `index-v1.html` is the proof it ran.
