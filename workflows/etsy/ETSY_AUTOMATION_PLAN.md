# Etsy listing automation: discussion, decisions and plan

Status as of 2026-10-30: **planning done, building has started.** Built so far:

- `etsy_mark_existing_listings.json`: adds the 13 green Etsy columns to every tab and marks the 853 listings already on Etsy (run for real on the original sheet).
- `Price In CAD` is an ARRAYFORMULA at rate 1.39 on all 30 tabs (Bilal set the rate and owns the QA).
- 2,618 rows were flagged `Etsy_ready_to_post = yes` (title, description, CAD price and photos present, on Shopify, not a duplicate SKU). Later products are flagged by hand by the team.
- `etsy_create_draft_listing.json`: creates one draft listing from a flagged row (dry run first, draft only, Bathtub, Sink, Marble+ Basin and Fireplace only). First live test on LSYP271 created the draft and all properties; photo uploads needed pacing and a second pass (added). Tags AI step and batches are not built yet.

The sections below are the original plan and decisions.

## 1. What this is

Automate creating and updating Etsy listings for Luna's Marblebee store from the same Google Sheet that drives the Shopify product sync ("Auto sync shopify 2026"), working the way the Shopify pipeline does: all product types at once, with one switch for creating new listings and updating existing ones.

On 2026-10-28 Luna handed over **the whole Etsy store**, not only the automation. She has no time to monitor it, so:

- Bilal makes the decisions. Luna is asked only when he truly cannot decide.
- Quality control of the Etsy store is Bilal's job, with a **weekly check**.
- If quality-control insight is needed, ask the other three people in the group chat: Raenia, Xing or Agen.
- Treat it as a whole project (listings, quality, prices, upkeep), not only a workflow.

## 2. Sources

- Two screen recordings by the person who lists on Etsy by hand: "Etsy upload demo 1" (https://youtu.be/lDTEseOmR1M, 21:49; narration and picture stop at about 10:05, the rest is black and silent) and "Etsy Upload demo 2" (https://youtu.be/6dMOKw4k6fE, 11:38, fully narrated). The product shown is LSYP464, a bathtub.
- Luna's answers to our questions, 2026-10-28 (section 4).
- The Shopify pipeline in `workflows/product-management/`, which already carries Etsy fields (section 5).

## 3. How the manual listing process works (from the videos)

1. **Photos:** the Drive folder is downloaded locally and the images are taken from its **logo sub-folder**, not the no-logo images.
2. **Category:** Etsy has no bar-top category, so the closest one is picked (Sinks & Basins, which is also used for bathtubs). The category decides which attribute fields Etsy shows. Some categories show no dimension fields at all.
3. **Title and description** come from the sheet (`Title_Etsy`, `Description_Etsy`). The product number goes at the end of the description as `Product No: <SKU>`, so customers and staff can find the product.
4. **Variations:** none. Quantity is 1 and SKU is the product number.
5. **Tags:** up to 13, each 1 to 20 characters. Etsy blocks a longer tag ("Tags must be between 1 and 20 characters"). By hand, a long tag is first shortened by merging words, then replaced with a similar phrase.
6. **Material:** picked from Etsy's suggestion (marble).
7. **Dimensions:** one unit is used for every listing (inches). Etsy shows either Width/Height/Depth or Length/Width/Height, depending on the category. Where Etsy has Depth, the sheet's Length goes into Depth. If the category has no dimension fields, a "Dimensions" line is added to the description in cm and inches (length x width x height).
8. **Shipping section:** weight in kg and packed size in cm, both from the sheet. The store already has a processing profile ("Made to order", 8 to 10 weeks) and a free-delivery profile; these are left as they are. (851 active listings use them.)
9. **Price:** Etsy takes CAD. The sheet's "Price In CAD" column is `=ROUNDUP(1.4*Z<row>)` where column Z is the USD price. The rate was 1.37 in the video and 1.4 in the sheet now; it changes daily and was updated by hand. Etsy shows each viewer a converted price, so a wrong rate is a real loss.
10. **Custom modifications box:** shown in the video as a required text box ("Custom modifications", limit 1024). **Luna has now said not to use it** (section 4).
11. Publish.

## 4. Luna's answers (2026-10-28)

| # | Question | Answer |
|---|---|---|
| 1 | Photo order | Studio shot first. Same as the sheet and Shopify, **not** the lifestyle-first order in the video. |
| 2 | CAD drawing | Needed on Etsy. |
| 3 | Custom modifications box | **No** custom modification box. |
| 4 | Price | Bilal decides. |
| 5 | Tags | None in the sheet. Generate **13 tags** with AI from one keyword per product. |
| 6 | Etsy category per tab | Bilal decides. Research the website, Shopify and the sheet. Ask her only if he cannot decide. |
| 7 | Dimensions | Bilal decides. |
| 8 | Material | All marble. |
| 9 | Go live | Yes. Quality control of the store is also Bilal's job, checked weekly. |

## 5. What the sheet already has

The Shopify pipeline's row builder already reads these Etsy columns, so the sheet is partly prepared:

`Title_Etsy`, `Description_Etsy`, `Tags_Etsy` (empty today), `who_made`, `when_made`, `Etsy_type`, `Etsy_quantity`.

Other columns used here: Product No., Picture, Care Guide URL, **CAD Drawing URL**, Material (stone name, e.g. "Rosa Eterea Pink"), Length/Width/Height in CM and Inch, Weight KG, `Price IN USD $ (Jack)`, `Price In CAD`.

Existing Shopify rules that matter for Etsy:

- Product photos are Image URL 1 to 8 at most. **Image URL11 is dedicated to the CAD drawing** (products have at most 8 photos, so it never collides with a real photo), so the CAD image goes last, after the product photos.
- A row with no valid price is never created (price guard). Do the same here.
- Updates to live products stay off unless a switch (`ALLOW_UPDATES_TO_PUBLISHED`) is turned on.
- Photo order can be disturbed by Shopify's asynchronous file changes (gallery order incident, Oct 2026). Etsy uses ranked image uploads, so verify the final order after uploading instead of assuming it.

## 6. Decisions made (proposed, to confirm when the build starts)

| Topic | Decision | Why |
|---|---|---|
| Price | Use the sheet's `Price In CAD` column exactly as it is. The automation does not compute the price. | The rate lives in one place that Luna's team already maintains. Add a weekly price-drift check, because the rate changes daily and a listing only changes price when its row is flagged. |
| Tags | AI generates 13 tags from the product title and type. Check each tag: 20 characters or fewer, no duplicates, exactly 13. Retry if invalid. A row that still fails is flagged and skipped. | Etsy rejects bad tags. Luna has no tags in the sheet. |
| Photos | Studio first, as in the sheet, then the other photos, then the CAD drawing last. Use the sheet's image links (they come from the same Drive folder, logo version). Verify they are the logo versions on a few products before the first run. | Luna's answers 1 and 2. |
| Custom box | Not used. | Luna's answer 3. |
| Material | Marble for every product. The sheet's stone name (Rosa Eterea Pink and so on) is not sent as the Etsy material, but can appear in the description. | Luna's answer 8. |
| Publishing | New listings go live. Start with a small test batch, send Luna the links, then run the rest. Update mode uses a switch like Shopify's. | Etsy charges a listing fee per listing, and a bad mapping on a live run is costly. |

## 7. Category mapping (the 30 sheet tabs)

The Shopify pipeline loops over these 30 tabs: 2026Instock, Animal, Balustrade, Bathtub, Bench, BookMatchedSlabs, Carving, Columns, DoorSurround, Exterior-Wall-Decoration, Fireplace, Fountain, Lamppost, Marble+ Basin, Marble+ ConsoleTable, MarbleMedallion, MarbleRangeHood, MarbleSlab, Moulding, OutdoorCornice, Planter, Sink, Stairs, Statue, StoneMosaicTile, Table, Vanity, Wainscoting, Wall-Coverings, WindowSurround.

Proposed Etsy **families**. These are not final category IDs. The exact category names and IDs must be taken from Etsy's live category tree through the API.

| Etsy family | Tabs | Confidence |
|---|---|---|
| Bathroom: Sinks & Basins | Bathtub, Sink, Marble+ Basin | High (the demo uses it for bathtubs) |
| Bathroom: Vanities | Vanity | High |
| Furniture: tables and benches | Table, Marble+ ConsoleTable, Bench | High |
| Fireplaces and hearth | Fireplace, MarbleRangeHood | Medium |
| Garden and outdoor | Fountain, Planter, Lamppost | Medium |
| Statues and sculpture | Statue, Animal, Carving | Medium |
| Architectural and building elements | Columns, Balustrade, Stairs, DoorSurround, WindowSurround, Moulding, OutdoorCornice, Exterior-Wall-Decoration, MarbleMedallion | Low: Etsy has no clean category, needs research |
| Wall and surface materials | Wainscoting, Wall-Coverings, StoneMosaicTile | Low |
| Raw stone slabs | MarbleSlab, BookMatchedSlabs | Low |
| Mixed stock sheet | 2026Instock | Not a product type. Take each row's own type instead of the tab. |

### Category mapping against Etsy's live tree (2026-10-28)

Etsy's seller taxonomy has 3,065 nodes. The mapping below uses real category IDs. Etsy has **no category** for fireplace mantels, columns, balustrades, stairs, range hoods, stone slabs or lampposts, so those get the nearest one and depend on title and tags for search. Test each tab with one draft before the first real run.

| Tab | Etsy category | ID | Confidence |
|---|---|---|---|
| Bathtub, Sink, Marble+ Basin | Home Improvement > Plumbing > Sinks & Basins | 11353 | High (the hand process uses it) |
| Vanity | Bathroom > Bathroom Vanities | 903 | High |
| Bench | Furniture > Living Room > Benches & Trunks > Benches | 12369 | High |
| Marble+ ConsoleTable | Console Tables & Cabinets > Console & Sofa Tables | 11355 | High |
| Table | Coffee Tables 12371 / End Tables 12372 / Kitchen & Dining Tables 977, picked by words in the title | | Medium |
| Fountain | Garden Decoration > Outdoor Fountains | 12240 | High |
| Planter | Planters & Pots > Outdoor Planters | 6061 | High |
| Moulding | Home Improvement > Moldings & Trim | 784 | High |
| DoorSurround, WindowSurround, OutdoorCornice | Moldings & Trim | 784 | Medium |
| Columns, Balustrade, Stairs, Wainscoting, Wall-Coverings, MarbleRangeHood | Moldings & Trim (nearest architectural category) | 784 | Low |
| Fireplace | Home Decor > Fireplace Screens (only fireplace category) | 12219 | Low |
| Lamppost | Lighting > Garden & String Lights > Garden Lights | 12410 | Medium |
| Statue, Animal | Home Accents > Statues (alternative: Outdoor Statues 2867) | 2869 | Medium |
| Carving | Art & Collectibles > Sculpture > Art Objects | 129 | Medium |
| Exterior-Wall-Decoration | Garden Decoration > Yard Art | 2784 | Low |
| StoneMosaicTile | Home Improvement > Flooring | 781 | Medium |
| MarbleMedallion | Home Improvement > Flooring | 781 | Low |
| MarbleSlab, BookMatchedSlabs | Raw Materials > Stones & Rocks > Stones & Pebbles (a craft-supply category; may need the supply flag) | 6651 | Low (fallback: Flooring 781) |
| 2026Instock | Not a product type: use each row's own type | | Needs a rule |

Only leaf categories were listed; whether Etsy accepts a non-leaf category must be checked with a draft.

### Dimensions per tab

Etsy only has Length, Width and Height (or Depth). The Shopify pipeline uses more kinds of measurement, which must not be lost:

| Tab group | Shopify measurements | Etsy handling |
|---|---|---|
| MarbleSlab, Wall-Coverings, StoneMosaicTile, MarbleMedallion, Exterior-Wall-Decoration, Wainscoting | length, width, thickness | Length/Width in Etsy fields where shown; thickness in the description line |
| Fireplace, MarbleRangeHood, Carving, WindowSurround | depth, width, height | Depth/Width/Height map directly |
| Moulding | length, height, thickness | Length/Height in Etsy fields; thickness in the description line |
| OutdoorCornice | length, height, projection | Length/Height in Etsy fields; projection in the description line |
| Stairs | width, height, diameter, inner opening diameter | Width/Height in Etsy fields; diameters in the description line |
| Balustrade | overall height, handrail height and width, baluster diameter and height, base/plinth width and height | Overall height in Etsy fields where shown; the rest in the description line |
| Other tabs | length, width, height | Direct |

**Which chosen categories have dimension attributes (checked against Etsy, 2026-10-28):**

- **Width, Height and Depth** (units cm, in, ft, m, mm, yd): 129, 784, 903, 977, 2869, 11353, 11355, 12219, 12369, 12371, 12372, 12410. Sheet Length goes into Depth.
- **Width and Length only:** 781 (Flooring).
- **No dimension attributes (description line only):** 12240 (Outdoor Fountains), 6061 (Outdoor Planters), 2867 (Outdoor Statues), 2784 (Yard Art), 6651 (Stones & Pebbles).
- **Required property "Craft type":** 784 and 781 (value "Home improvement"), 6651 (Gardening / Kids' crafts / Mosaic making / Terrariums). These behave like craft-supply categories, so the listing may need the supply flag; confirm with a draft.
- **Material:** most categories have a "Materials" list (569 values); 129 uses "Medium", 6651 uses "Stone type", 11353 has a short "Material" list (7 values) that may not include marble. If marble is not offered, put it in the description and tags.
- Etsy's packed-size and weight fields (item length, width, height, weight) are separate from these attributes and apply to every listing.

**Etsy property IDs (read from the live API, 2026-10-28; same in every category that has the field):**

| Field | property_id | Notes |
|---|---|---|
| Width | 47626759898 | inches scale_id **5** |
| Height | 47626759834 | inches scale_id **5** |
| Depth | 54142602037 | inches scale_id **5** (sheet Length goes here) |
| Length | 47626759838 | only on 781 (Flooring), scale_id 5 |
| Materials | 148789511893 | most categories |
| Medium | 570246213483 | category 129 only |
| Stone type | 99837394284 | category 6651 only |
| Material | 570246212865 | category 11353 only |
| Craft type | 47626759760 | required on 781, 784, 6651 |

- **Marble is value_id 171** in every material list above.
- **Craft type values:** Home improvement = 560 (781 and 784). For 6651: Gardening 1236, Kids' crafts 562, Mosaic making 571, Terrariums 591 (use 571, the nearest to slabs).
- Categories that have no dimension fields (2784, 2867, 6061, 6651, 12240) still take Materials (or Stone type).
- Re-read the properties from Etsy if a category is changed; do not copy IDs between categories.

Rule for every product: units in inches in Etsy's fields. The description line carries the full specification in cm and inches. Weight in kg and packed size in cm go in the shipping section.

## 8. Implementation plan

**Phase 0: access and groundwork**

1. Etsy developer app (API keystring) and OAuth for Luna's shop with permission to read and write listings. Confirm whether the approval needs a waiting period. Check the existing "Etsy token" workflow, if there is one, before creating a new one.
2. Shop ID, the existing shipping/delivery profile ID, and the processing profile ("Made to order, 8 to 10 weeks"). Etsy now also wants a readiness state on a listing; confirm the current fields against Etsy's docs before building.

**Phase 1: category map**

1. Pull Etsy's category list and the fields each category shows (the taxonomy and property endpoints).
2. Fill in section 7 with real category IDs, one per tab.
3. Create one **draft** listing per tab to confirm Etsy accepts the category and the fields. Ask Luna only about the tabs that cannot be settled.

**Phase 2: build the workflow** (reuse the Shopify sync patterns)

1. Read the sheet tabs in a loop, in two steps to keep memory low.
2. A filter with create and update switch (create mode returns null for updates and the reverse), plus the zero-price guard.
3. Build the listing: title, description with `Product No:` line, tags (AI step, then validation), material marble, quantity 1, SKU, price from `Price In CAD`, weight and size, who_made and when_made from the sheet, dimensions per section 7.
4. Create draft, upload images in order (studio first, CAD last), verify the final order, then publish.
5. Write back to the sheet: Etsy listing ID, listing URL, status, error. Mark a row published only after Etsy confirms.
6. A run lock and a cap on listings per run, like the Pinterest workflows. Respect Etsy's rate limits (check the current limits).

**Phase 3: test, then go live**

1. Test batch: one product per category family. Check each on Etsy (photos, order, category, dimensions, tags, price).
2. Send Luna the links. After her approval, run the rest in batches.
3. Update mode: leave updates to live listings off until a deliberate run.

**Phase 4: weekly quality control**

A weekly automatic report to Bilal (and a copy to the people helping, if wanted):

- Listings on Etsy versus rows marked ready in the sheet.
- Listings with missing photos, wrong photo order, no CAD drawing, or tags not 13.
- Price drift: Etsy price versus `Price In CAD` now.
- Rows flagged for tag, category or price problems.
- Etsy errors and anything that failed to publish.

### Etsy API app (found 2026-10-28)

- Developer dashboard has a **personal app** "marblebee-inventory-automation" with **Personal Access** status (own shop only), limits **5 requests per second and 5,000 per day**. It is not connected to n8n yet.
- A second app, **"marblebee-catalog-manager", is listed as Banned.** Find out why before building (check email from Etsy developer support), and avoid whatever caused it.
- The 5,000-per-day limit matters: one listing is roughly a create call, several image uploads and property updates, so only a few hundred listings per day are possible. Plan batches of listings over several days and count calls.
- **Header format (confirmed by Etsy's own error, 2026-10-28):** every call needs `x-api-key: <keystring>:<shared_secret>`, both parts joined by a colon. The keystring alone returns 403 "Invalid API key: should be in the format 'keystring:shared_secret'". The OAuth2 credential in n8n (named after the shop) is connected, but it only supplies the bearer token, so the `x-api-key` value must be set on each HTTP node.
- Never put the shared secret in the repo or in chat. In the repo keep a placeholder. In n8n, prefer n8n Variables (if the plan has them) or one Config node in the workflow that nobody exports or commits with the real value.

### Connection verified (2026-10-28)

OAuth connected in n8n (generic OAuth2 API credential, PKCE, scopes `listings_r listings_w shops_r`, authorized by the account behind lunaruiz715@gmail.com). A read-only test returned:

- Shop **MarblebeeDesign**, shop_id **64324681**, currency **CAD**, **852 active listings**, user_id 914417498. (The profile page shows 0 sales; worth tracking in the weekly check.)
- Shipping profile "Free Delivery", id **305991461140**, ships from Canada.
- Processing profile (readiness state) made_to_order, id **1485857866342**.
- Top-level categories include Home & Living (891), Art & Collectibles (66), Craft Supplies & Tools (562); the useful ones sit deeper in the tree.

**Consequence: the shop already has 852 listings.** Create mode must not duplicate them and update mode must match each sheet row to its existing listing. Match on SKU (= product number). Reading all listings takes about 9 calls (100 per page).

The Marblebee business has two Etsy accounts; this credential covers only MarblebeeDesign. Name any further credential after its shop.

### Snapshot of existing listings (read-only run, 2026-10-28)

Etsy returned 852 active, 1 draft, 2 inactive, 0 sold out, 0 expired; all pages fetched, no errors.

- **Every active listing has exactly one SKU** (SKU = product number, e.g. LSYP311). No listing without a SKU, none with several.
- **3 SKUs appear on two active listings each:** LSBLD533 (4530561892, 4512963494), XHYP127 (4513535808, 4513535814), LSYP270 (4525674018, 4525675242). Check by hand before automation touches them (duplicate fees, or a deliberate relist?). The match logic must flag a SKU that matches more than one listing, never pick one silently.
- **The store only has two categories today:** 12219 Fireplace Screens (492 listings, used for fireplaces) and 11353 Sinks & Basins (360, used for bathtubs). So 28 of the 30 sheet tabs have no Etsy listings yet; the first runs for those tabs are all creates. This confirms the hand process uses 12219 for fireplaces.
- **Identical on all 852:** CAD, quantity 1, one shipping profile (305991461140), one processing profile (1485857866342), who_made `collective`, when_made `made_to_order`, no variations, not supply, weight stored in grams (e.g. 800000 g = 800 kg) on every listing.
- **Prices (CAD):** min 6,659, median 12,096, max 67,062.
- **Photos per listing:** 4 to 11 (most 6 or 7; 117 have 8 or more). Our plan sends at most 8 product photos plus the CAD drawing.
- **Tags:** 424 listings have 13, 358 have 12, 68 have 9, 2 have 10; none over 20 characters. **The tags are identical across a whole category** (all fireplaces share the same 13, all bathtubs the same 12), i.e. copied by hand, not written per product. Per-product AI tags (Luna's answer 5) would be a change from today; flag it for the weekly check and decide whether to retag existing listings or leave them.
- The listing-level "materials" field is empty on all 852; the material is probably set through the category attributes instead. Not confirmed; check one listing before relying on it.
### Comparison with the sheet (read-only run, 2026-10-28)

Read 855 Etsy listings (852 active, 1 draft, 2 inactive) against 8,770 sheet rows with a Product No. in 28 tabs (Animal and Lamppost are empty).

- **Matching by SKU works:** 847 sheet rows match an active Etsy listing. Only 2 active listings have no sheet row: LSBLD586 (4535475567) and LSBLD475 (4507198807). Decide whether to keep them (orphans) or add them to the sheet.
- **Prices already follow the sheet:** of 847 matched, 831 are exact, 1 is within 1 CAD, 15 are off. Biggest gaps: LSYP250 (sheet 29,798, Etsy 21,756), LSYP234 (sheet 21,756, Etsy 26,373; note LSYP250's Etsy price equals LSYP234's sheet price, so possibly swapped or stale), LSYP462 and LSYP463 (about 1,000 each, opposite directions), LSBLD858 (-828), LSBLD567 (+704), LSBLD780 (+456), LSBLD856 (+358), LSBLD584 (+214), LSBLD580 (-207); the other five are under 100. Nothing has a blank sheet price.
- **Where the listings are:** Bathtub 358 of 659 rows on Etsy (301 not), Fireplace 489 of 970 (481 not). **Every other tab has 0 listings on Etsy**, including all rows of Sink, Vanity, Table, Statue and so on.
- **Ready_to_post counts per tab (the Shopify flag):** MarbleRangeHood 313, Moulding 174, Sink 139, Fireplace 129, MarbleSlab 64, Marble+ Basin 52, Marble+ ConsoleTable 50, Bathtub 43, Wainscoting 30, Balustrade 25, DoorSurround 24, Vanity 24, WindowSurround 24, Fountain 21, Table 21, Wall-Coverings 6, all others 0. About 1,139 rows in total. The Shopify pipeline resets this flag to "no" after it creates a product, so Etsy needs its own status column in the sheet (see open items).
- **29 SKUs appear in two tabs:** XHHP24 to XHHP31 in both 2026Instock and Planter; LSSD001 and following in both Bench and MarbleMedallion (the Bench tab seems to hold medallion rows; 14 more not listed here). Create each SKU on Etsy once, whatever the tab.
- **Not active on Etsy but in the sheet:** LSYP349 (draft), LSYP257 and LSYP272 (inactive). Never create a second listing for these; decide whether to reactivate.
- **Same SKU on two active listings:** LSBLD533, XHYP127, LSYP270 (see above).

### Decisions by Bilal (2026-10-28, after the comparison)

- **Create and update work like Shopify:** create mode makes a small number of new listings per category per day (default 3 per tab per run; 4 or 5 per tab for the first live test), update mode can update any number of flagged rows. One create/update switch, same meaning and same Settings node as the Shopify pipeline.
- **First goal: mark the existing Etsy listings on the sheet** so they are never created again. Built as `etsy_mark_existing_listings.json` (dry run first, writes only after `dryRun` is set to false).
- **Only rows marked ready are created.** Many products are live on Shopify but not on Etsy, so the ready flag for Etsy must be separate from Shopify's.
- **Why `Ready_to_post` cannot be reused for Etsy:** in the Shopify filter, a ready row that already has a Shopify URL and no `Fields_to_update` goes to the "mark false" path, which sets `Ready_to_post` back to "no" and changes nothing on the product. Setting it to yes on live Shopify products would therefore be cleared by the next Shopify run (and could be cleared before Etsy sees it).
- **New columns at the end of each tab** (added after the last header; the sheet is widened if needed; Shopify's workflow never reads them): `Etsy_listing_id`, `Etsy_url` (public page), `Etsy_admin_url` (Shop Manager editor link, like Shopify_Update_product_URL), `Etsy_status` (live, draft, inactive, duplicate_on_etsy, later also created/error), `Etsy_error`, `Etsy_created_at`, `Etsy_last_modified` (both from Etsy), `Etsy_price_cad` (price on Etsy at the last sync, for the weekly drift check), `Etsy_category_id`, `Etsy_image_count`, `Etsy_synced_at` (when the automation last refreshed the row), `Etsy_fields_to_update`, `Etsy_ready_to_post`. Static values (id, URLs, created date, category) are never overwritten if they differ (reported as conflicts); changing values (status, last modified, price, image count, synced at) are refreshed on every run.
- **Admin link format (confirmed from a real listing, 2026-10-28):** `https://www.etsy.com/your/shops/me/listing-editor/edit/<listing_id>` (Etsy appends `#media` or similar section anchors; they are left out). `me` means whichever shop the browser is signed in to, so with two Etsy accounts the person must be signed in to MarblebeeDesign.
- **Duplicate SKUs across tabs** (29): skip until the owning tab is decided; do not create them. Cause of Bench holding medallion SKUs is unknown.

## 9. Open items

1. **Etsy API access:** do we already have an API key and shop authorization? (Blocks everything.)
2. **Scope:** weekly store quality control is more than the original automation job. Confirm it is covered by what was agreed on time and cost.
3. **Photos:** check on a few products that the sheet's image links point to the logo versions.
4. **Description dimensions line:** confirmed as both cm and inches; confirm the exact wording with the person who lists.
5. **Categories:** the Low-confidence rows in section 7 need research.
6. **Weekly price check:** decide who acts when the exchange rate drifts (the sheet's formula uses 1.4 now).
7. **Etsy status columns in the sheet:** propose adding Etsy_ready, Etsy_listing_id, Etsy_url, Etsy_status and Etsy_error to each tab (sheet changes need confirmation first). Ready_to_post cannot be reused, because the Shopify run resets it.
8. **Scale:** the sheet has about 8,770 rows against 852 listings on Etsy. Decide which rows go first (for example Ready_to_post rows, about 1,139) before any bulk create; every listing costs an Etsy fee.
9. **Bench tab and duplicate SKUs:** ask Luna's team why Bench holds medallion SKUs, and decide which tab owns each of the 29 duplicated SKUs.

## 10. Risks

- Etsy has no clean category for architectural stone, so listings may be mis-categorized and found less in search.
- A wrong exchange rate means every price is wrong at once.
- Each new listing costs an Etsy fee, so a bad batch is expensive. Hence the test batch.
- Photo order can be wrong after upload; always verify, as with Shopify.
- Etsy API limits and approval times may slow the first run.
