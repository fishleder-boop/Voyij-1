# Voyij — Made In Alaska Collection Page

**Route:** `/madeinalaska`
**File:** `index.html` (standalone prototype; see integration notes below)

---

## What's Built

A full-page, production-quality collection page for the Made In Alaska program. Includes:

| Section | Description |
|---|---|
| **Hero** | MIA emblem SVG, H1, subhead, CTAs, trust pill |
| **Filter Bar** | Sticky search + category chips + sort dropdown + Products/Stores toggle |
| **About / Trust** | 3 info cards explaining the program (placed before listings) |
| **Stories** | 3 editorial story cards tagged "Made In Alaska" |
| **Featured Stores** | 3 curated store cards with permit numbers |
| **Product Grid** | 12 mock products, each with MIA badge, price, store, city, View + Add to Cart |
| **Stores Directory** | 6 store cards shown when toggle = Stores |
| **Header / Footer** | Faithful Voyij recreations; replace with Yo!Kart partials |

---

## Yo!Kart Integration

### 1. Route / CMS Page

In Yo!Kart admin → **SEO URLs** → create:

```
/madeinalaska  →  CMS Page ID: [your page id]
```

Or create a custom controller in `YoKart/controllers/` with a `madeinalaska` action.

### 2. Header & Footer

Replace the inline header/footer in `index.html` with your Yo!Kart template partials:

```smarty
{* Top of file *}
{include file="header.tpl"}

{* Bottom of file *}
{include file="footer.tpl"}
```

### 3. Product Data

Replace `MOCK_PRODUCTS` in the `<script>` block with a real API call:

```javascript
// Option A — server-rendered JSON blob (recommended for Yo!Kart):
const MOCK_PRODUCTS = {literal}{$madeInAlaskaProducts|json_encode}{/literal};

// Option B — async fetch:
fetch('/api/products?madeInAlaska=true')
  .then(r => r.json())
  .then(({ products }) => {
    document.getElementById('product-grid').innerHTML = products.map(renderProduct).join('');
    document.getElementById('count-num').textContent = products.length;
  });
```

**Required product fields:**
```javascript
{
  id:             number,
  name:           string,
  price:          number,        // USD
  category:       string,        // "food" | "jewelry" | "gifts" | "home" | "art" | "apparel" | "beauty"
  store:          string,
  city:           string,
  image:          string,        // CDN URL (use responsive srcset if available)
  isMadeInAlaska: boolean,
  permitNumber:   string,        // e.g. "MIA-2024-0142"
  slug:           string,        // /products/{slug}
  storeSlug:      string,        // /business/{slug}
}
```

### 4. Store Data

Replace `MOCK_STORES` similarly:

```javascript
const MOCK_STORES = {literal}{$madeInAlaskaStores|json_encode}{/literal};
```

**Required store fields:**
```javascript
{
  id:           number,
  name:         string,
  city:         string,
  permitNumber: string,
  permitStatus: "active" | "expired",
  logo:         string,
  cover:        string,
  blurb:        string,
  slug:         string,
  productCount: number,
}
```

### 5. Permit Status API

Optionally validate permit status on page load:

```javascript
const permitNumbers = MOCK_STORES.map(s => s.permitNumber).join(',');
fetch(`/api/permit-status?permitNumbers=${permitNumbers}`)
  .then(r => r.json())
  .then(statusMap => {
    // statusMap: { "MIA-2024-0142": "active", "MIA-2024-0089": "expired" }
    // TODO: Re-render or suppress products with expired permits
  });
```

### 6. Stories / Blog

Replace `MOCK_STORIES` with posts tagged `made-in-alaska`:

```javascript
fetch('/api/blog?tag=made-in-alaska&limit=3')
  .then(r => r.json())
  .then(({ posts }) => {
    document.getElementById('stories-container').innerHTML = posts.map(renderStory).join('');
  });
```

### 7. Fonts

`DM Sans` is loaded from Google Fonts as a close match for `Harmonia Sans Pro`. If Harmonia Sans Pro is already loaded by the Yo!Kart theme, remove the Google Fonts `<link>` and update:

```css
--font-body: 'Harmonia Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```

### 8. Made In Alaska Emblem

The `<svg class="mia-emblem">` in the hero is a faithful recreation of the program mark for prototyping purposes. Before launch, **replace this with the official licensed Made In Alaska mark image**:

```html
<!-- Replace the <svg class="mia-emblem"> with: -->
<img
  class="mia-emblem"
  src="/assets/images/made-in-alaska-mark.png"
  srcset="/assets/images/made-in-alaska-mark@2x.png 2x"
  alt="Official Made In Alaska program mark"
  width="220" height="220"
/>
```

Obtain the official mark from the [Alaska Division of Economic Development](https://www.commerce.alaska.gov/web/ded/MADEINALASKA.aspx).

---

## Design System Reference

| Token | Value | Usage |
|---|---|---|
| `--teal` | `#19B3A6` | Primary action, links, badges |
| `--teal-dark` | `#128F83` | Hover state |
| `--teal-light` | `#E4F6F5` | Subtle backgrounds |
| `--navy` | `#0B3260` | Headings, hero bg |
| `--navy-mid` | `#1A4475` | Hero gradient stop |
| `--gold` | `#C8962A` | MIA emblem accent |
| `--font-display` | Playfair Display | H1, section titles |
| `--font-body` | DM Sans | All body copy |

---

## Category Mapping

The filter chips use these `data-category` values — ensure your backend maps product categories accordingly:

| Chip label | `data-category` value |
|---|---|
| Food & Pantry | `food` |
| Jewelry | `jewelry` |
| Gifts | `gifts` |
| For the Home | `home` |
| Art & Crafts | `art` |
| Apparel | `apparel` |
| Health & Beauty | `beauty` |
