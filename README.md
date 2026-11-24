# ironix-size-finder
# IRONIX Size Finder (prototype)

Part 2 of the IRONIX web development assessment. A single-file demo of a "Find my size" tool for The Keeper ribbed tank.

**Live demo:** https://ishan090hdbsfjdsdsfwf.github.io/ironix-size-finder/

Sample stock data is fake, and nothing here was tested on a live Shopify theme.

## Two proposals

### 1. Size and fit finder (built)
- **Problem:** The product page has a size chart where L (chest 40–42) and XL (40–44) overlap, and the copy says both "body-hugging" and "regular athletic fit", plus "size up by one" for a roomier fit. Shoppers have to work out their size themselves. IRONIX's return policy charges Rs. 150 for exchanges after the first, so a wrong size costs the business and the customer.
- **How it works:** The customer enters a chest measurement (in or cm), picks a fit preference and a colour, and gets one recommended size, with a clear message when they sit between two sizes or the size is sold out.
- **Why IRONIX:** A single product line with four sizes and a 48-hour return window makes size confidence the main purchase risk. I have no IRONIX return data, so I can't quantify the effect; it is a hypothesis to test.

### 2. Mix-and-match combo builder (not built)
- **Problem:** Combos are 10 separate products ("The Keeper Combo" × 10) and the tiered offer (the announcement bar promises more savings at 2 and 3 items) is baked into each. Customers can't freely choose colours or sizes per item.
- **How it works:** A builder page where the customer picks 2 or 3 tanks (colour and size for each) and sees the tier price update live. It adds the items to the cart and an automatic discount or Shopify Function applies the tier.
- **Why IRONIX:** The homepage pushes "One is never enough". A builder supports that goal and would also let IRONIX retire the 10 near-duplicate combo products, which helps the SEO findings in the audit.

## Run it
No build step. Either:
- open `index.html` in a browser, or
- `python3 -m http.server 8000` in this folder and visit http://localhost:8000

## How the logic works
- Size chart data lives in `SIZES` (from the product page). Sample stock is in `PRODUCTS` and is **fake**.
- `recommend(chestInches, fit)` ranks sizes by distance from each range's midpoint. This resolves the L/XL overlap without hard-coding it.
- If the two closest sizes are within 0.25 in, the customer is "between sizes": body-hugging picks the smaller, roomier picks the larger.
- Roomier otherwise moves up one size (the site's own note), capped at XL.
- `pickInStock()` finds the nearest in-stock size if the recommendation is sold out.

## Edge cases handled
- Empty, non-numeric or zero input shows a clear error.
- Chest below 35 in or above 45 in gets no recommendation and points to support.
- 35–36 in and 44–45 in get a recommendation with an "edge of our range" note.
- Between two sizes (for example 38 in or 41.5 in).
- Recommended size sold out in the chosen colour; all sizes sold out in a colour.
- Changing colour after a result re-checks stock; changing fit re-runs the result.
- cm input is converted to inches.

## Shopify integration
1. Create `sections/size-finder.liquid` (or a theme app block) and render it inside `main-product.liquid`, below the variant picker.
2. Replace the sample data with real data:
   - Size chart: a product metafield such as `custom.size_chart` (JSON), editable in the admin.
   - Stock: output `product.variants` as JSON in Liquid (`size`, `color`, `available`, `id`).
3. On a recommendation, select the matching variant radio in the theme and dispatch a `change` event, so the price, image and Add to cart button update with the theme's own logic.
4. For "Add to cart", call `POST /cart/add.js` with the variant id, or just click the theme's existing button.

**Dependencies and limitations**
- Variant selector markup differs by theme, so step 3 needs theme-specific selectors; I tested nothing on a live theme.
- Availability here is per variant (`available`), not exact quantity, unless inventory is exposed some other way.
- The chart is chest-only and it is unclear whether the numbers are body or garment measurements. IRONIX should confirm this and fix the L/XL overlap at the source.
- No analytics are wired in. In production, log recommended vs purchased size to see whether it reduces exchanges.

## 3–5 minute walkthrough (suggested order)
1. Show the product page problem (size chart overlap, fit wording).
2. Enter 39 in, body-hugging, Black: M, in stock.
3. Enter 38: between S and M message; flip to roomier and see M.
4. Switch to White with M recommended: sold out, nearest in stock suggested.
5. Enter 30 and "abc": out of range and validation message.
6. Switch to cm (for example 99 cm).
7. Explain the Shopify integration steps above.

## AI disclosure
I used Claude (Anthropic) to help draft the audit, the proposals and the code. I reviewed everything, tested the recommendation logic against edge-case inputs, and can explain each part. Product facts (size chart, return policy, prices) come from the public ironix.in pages.
