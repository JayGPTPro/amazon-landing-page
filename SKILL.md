---
name: amazon-landing-page
description: Build a premium landing page from any Amazon product URL. Extracts real product data (title, images, features, reviews, price) via Chrome MCP and generates a complete, beautiful landing page with scroll animations, real reviews, FAQ, email capture, and mobile-optimized CTA. Use when the user says "build a landing page", "create a product page", "amazon landing page", or provides an Amazon product URL.
argument-hint: "<amazon-product-url>"
user-invocable: true
---

# Amazon Product Landing Page Builder

You are about to build a premium landing page for the Amazon product at **$ARGUMENTS**.

## Requirements

- **Chrome MCP is required.** Test it immediately. If unavailable, tell the user to enable it.
- The landing page will be a single HTML file with Tailwind CDN (no build step needed).
- All product data must be extracted from the real Amazon page. Never invent or fabricate data.

## Process

### Phase 1: Setup

1. Create a project folder: `amazon-landing-page-[product-name]` in the current directory.
2. Create an `images/` subfolder inside it.

### Phase 2: Extract Product Data via Chrome MCP

1. Open the Amazon product URL in Chrome MCP.
2. Wait for the page to load (3 seconds).
3. Extract the following using JavaScript execution:

```javascript
// Product basics
{
  title: document.getElementById('productTitle')?.textContent?.trim(),
  price: document.querySelector('.a-price .a-offscreen')?.textContent?.trim(),
  rating: document.querySelector('#acrPopover')?.title || document.querySelector('.a-icon-alt')?.textContent,
  reviewCount: document.querySelector('#acrCustomerReviewText')?.textContent?.trim(),
  brand: document.querySelector('#bylineInfo')?.textContent?.trim(),
  badges: document.querySelector('#acBadge_feature_div')?.textContent?.trim() // Amazon's Choice, Best Seller, etc.
}
```

```javascript
// Feature bullets
Array.from(document.querySelectorAll('#feature-bullets .a-list-item'))
  .map(el => el.textContent?.trim())
  .filter(t => t && t.length > 10)
```

```javascript
// High-res image URLs
Array.from(document.querySelectorAll('#altImages .a-button-thumbnail img'))
  .map(img => img.src.replace(/\._.*_\./, '._SL1500_.'))
  .filter(s => s.includes('images/I/'))
```

4. Scroll to the reviews section and extract REAL reviews:

```javascript
// Only 4-5 star verified reviews
Array.from(document.querySelectorAll('[data-hook="review"]')).map(r => ({
  stars: parseFloat(r.querySelector('[data-hook="review-star-rating"] .a-icon-alt')?.textContent || '0'),
  body: r.querySelector('[data-hook="review-body"] span')?.textContent?.trim(),
  author: r.querySelector('.a-profile-name')?.textContent?.trim(),
  date: r.querySelector('[data-hook="review-date"]')?.textContent?.trim(),
  verified: !!r.querySelector('[data-hook="avp-badge"]')
})).filter(r => r.stars >= 4 && r.body && r.body.length > 20 && r.verified)
```

### Phase 3: Download Images

Download all product images using curl:
```bash
curl -sL "[image-url]" -o images/product-1.jpg
```

### Phase 4: Build the Landing Page

Create `index.html` with the following sections (IN THIS ORDER):

1. **Fixed Navigation Bar**
   - Brand name on the left
   - Section links (Features, Reviews, FAQ) in the center (hidden on mobile)
   - "Buy on Amazon - $[price]" CTA button on the right
   - Glassmorphism background (backdrop-blur)

2. **Hero Section**
   - LEFT: Product image gallery with thumbnails (clickable to change main image)
   - RIGHT: Badge (Amazon's Choice / Best Seller if available), product title (rewritten as catchy headline), short description, star rating with review count link, price with "FREE Prime Delivery", trust badges (30-Day Returns, Ships via Amazon, 1-Year Warranty), CTA button with pulse animation
   - Both sides animate in from left/right on load

3. **Social Proof Bar**
   - Animated counter for customer count, average rating, key specs
   - Counter animates when scrolled into view

4. **Features Section** (id="features")
   - 6 feature cards in 3-column grid
   - Each card: icon, title, description (extracted from Amazon bullets, rewritten to be concise)
   - Staggered scroll reveal animations

5. **What's in the Box Section**
   - Left: product image
   - Right: checklist with green checkmarks of everything included
   - Items animate in one by one on scroll

6. **Image Gallery**
   - 2-3 column grid with hover zoom effect
   - All product images

7. **Email Capture Section**
   - "Join our VIP list" messaging
   - Email input + "Join VIP List" button
   - On submit: saves to Google Sheet (if configured) + shows success message with Amazon link
   - Include setup comments in the code for Google Form integration:
     ```
     const GOOGLE_FORM_URL = ''; // Seller fills this in
     const EMAIL_FIELD_ID = ''; // Seller fills this in
     ```

8. **Real Reviews Section** (id="reviews")
   - ONLY use reviews extracted from Amazon. NEVER invent reviews.
   - Show only 4-5 star verified reviews
   - Display author name, star rating, date, "Verified" badge
   - Link to "Read all reviews on Amazon"

9. **FAQ Section** (id="faq")
   - 5 relevant questions with accordion toggle
   - Generate FAQs based on the product type and features
   - Common patterns: battery life, compatibility, warranty, size/weight, setup

10. **Final CTA Section**
    - Gradient card with headline, subtext, and large CTA button

11. **Footer**
    - Disclaimer: "Independent product page. [Brand] is a registered trademark. Purchase fulfilled by Amazon.com."

12. **Sticky Mobile Bar** (visible only on mobile after scrolling)
    - Price + "Buy on Amazon" button fixed to bottom

### Design System

**IMPORTANT: Do NOT use a generic dark theme. Adapt the design to match the brand and product.**

Before choosing colors and style, look at:
- The brand's own website and packaging colors
- The product category (tech = sleek/minimal, baby products = soft/warm, outdoor = earthy/rugged, beauty = elegant/clean)
- The product images' dominant colors

Build a color palette that feels like it belongs to this specific brand and product. The landing page should look like the brand made it themselves, not like a generic template.

```
Base design principles:
- Typography: Inter (Google Fonts) or a font that matches the brand feel
- Border radius: rounded-2xl (cards), rounded-full (buttons, badges)
- Animations: scroll reveal (fade up), stagger delays, counter animation, pulse CTA glow
- CTA button: use a color that contrasts well and stands out
- Stars: amber-400 (#fbbf24)
```

### Technical Requirements

- Single HTML file, no build step
- Tailwind CDN via `<script src="https://cdn.tailwindcss.com">`
- Custom Tailwind config for brand colors
- All CSS animations via `<style>` tag
- All JavaScript at the bottom in `<script>` tag
- Responsive: mobile-first, works on all screen sizes
- All Amazon links should use target="_blank"
- Image gallery with JavaScript thumbnail switching

### Phase 5: Open and Verify

1. Open the HTML file in the browser automatically: `open index.html`
2. Tell the user the page is ready and list what was included
3. Remind them about the Google Form setup for email capture:
   - "To collect emails, create a free Google Form with one email field"
   - "Get the form action URL and field entry ID"
   - "Paste them into the GOOGLE_FORM_URL and EMAIL_FIELD_ID variables in the HTML"

### Important Rules

- NEVER fabricate or invent reviews. Only use reviews extracted from the Amazon page.
- If there are no 4-5 star verified reviews, skip the reviews section entirely and tell the user.
- All product data (title, price, features, images) must come from the real Amazon page.
- The headline in the hero should be a rewritten, catchy version of the product title (not the full Amazon title).
- Keep the FAQ answers helpful and accurate based on the product features.
- The landing page should look premium and professional, not like a template.
