# Classybath.com to Shopify: research and findings

Date: 2026-11-06. Research only, nothing was built or changed.
Question from Luna: move classybath.com to Shopify without losing photos or products, and later use it to feature products with a 3D display. Also: some products on Marblebee are no longer sold and must be deleted on Marblebee and Etsy (Raenia will send the list).

## 1. Short answer

It is doable and fairly simple, because classybath.com is a small site. There is no clever "transfer" button: the pages have to be rebuilt as Shopify products, and the photos copied across. Nothing is lost if the old addresses are redirected.

Recommended plan: a **separate new Shopify store** for Classybath, filled by a one-time import, with the old pages redirected to the new ones. 3D can be added product by product afterwards.

## 2. What classybath.com is today (checked on the live site)

| Item | Finding |
|---|---|
| Type of site | A hand-written static website: plain `.html` files on an Apache server. No WordPress, no store platform, no database. |
| Pages | 219 pages in the sitemap: about 173 product pages, 23 blog articles, 23 other pages (home, contact, category pages). |
| Product pages | Each has a title, description, dimensions (cm and inch), material and weight, and about 3 photos. **No price and no cart**: customers use a contact/inquiry form. |
| Photos | Roughly 500 (estimate: about 3 per product page, stored in `/img/...` folders). A full count needs a scan of every page. |
| Products | Bathtubs (oval, rectangular, rolled rim, Japanese soaking, riverstone, carving, precious stone, stone, granite, travertine), pedestal sinks, marble basins, tabletops. Product numbers on 59 pages follow Marblebee's style (LSYP001 to LSYP031 and XHYP01 to XHYP33). The other pages use plain names such as `stone-bathtub4` or `marble-sink14`. |
| 3D | None today. |
| Marblebee overlap | None found in Marblebee's public product list for the numbered products (LSYP001 to 031, XHYP01 to 33). It looks like a separate, older catalogue. Marblebee's bathtub numbers start much higher. This should be confirmed against the sheet. |
| Tracking | Google Tag Manager is installed (an ID to keep or reuse). |

Because there is no export file, the content has to be collected from the pages themselves (a scan of the site), not downloaded.

## 3. How to move it, step by step

1. **Decide the structure** (see section 5): one new Shopify store for Classybath.
2. **Collect everything** from the live site with a script: title, description, dimensions, material, weight, photos, and the old page address for each product. Keep a copy of the originals (a backup of the whole site).
3. **Build a product list** (a spreadsheet in Shopify's import format): one row per product, photo links included. Shopify downloads the photos from the links, so nothing has to be uploaded by hand. Prices stay empty or "contact us" if Classybath does not show prices.
4. **Import** into the new store, then check a sample by eye and count products and photos against the old site.
5. **Blog and pages:** the 23 articles and the other pages are moved as Shopify blog posts and pages (copied across, or rebuilt).
6. **Redirects:** make a list "old address, new address" for every page and import it in bulk (Shopify: Content, Menus, URL redirects, Import). This keeps Google rankings and old links working. Note: Shopify cannot redirect `page.html` to `page` for the same page, so each old `.html` address needs its own mapping to the new product address.
7. **Point the domain** classybath.com to Shopify (A record `23.227.38.65`, `www` CNAME `shops.myshopify.com`, then connect it in Shopify Domains). It can take up to 48 hours to go fully live. Keep the old site running until the check passes.
8. **Final check** after launch: test the redirects, the product count, the photo count, search results, and the contact form.

## 4. 3D display later

- Shopify supports 3D models on product pages in two formats: **GLB** and **USDZ**. Shopify pairs them: iPhones open the AR view with USDZ, other devices use GLB.
- Maximum size per 3D file: 500 MB. Files over 15 MB load slowly, so keep models small (a few MB is ideal).
- Each product can hold up to 250 items (images, videos and 3D models together). Stores have a total limit of hosted videos and 3D models per plan (250 on Basic).
- Customers can rotate the model on the product page and place it in their room with AR on a phone. Most modern Shopify themes show it out of the box, which still needs a quick test in the chosen theme.
- The 3D files have to be made or bought separately (a 3D artist or a scan of the product). Shopify does not create them.
- Because Classybath is a **separate store**, 3D products can be added there without touching Marblebee.

## 5. One store or two?

| Option | Verdict |
|---|---|
| **Separate new Shopify store for Classybath** | Recommended. Clean brand, own domain, own theme, own products. Costs one more Shopify subscription (check the current price on Shopify's pricing page; sources I found quoted $29 to $39 a month for Basic). |
| Use the existing Marblebee store with a second domain | Not recommended. Shopify does not natively show two different brands with their own domains from one store. It gets messy and risks mixing the two sites. |

Our automation (the Google Sheet and n8n flows) works on the Marblebee store. A second store would need its own copy of those flows later, if Luna wants the same automation there.

## 6. Risks and things to decide

1. **Prices:** Classybath shows none. Does Luna want to sell online (prices, cart, checkout) or keep it as an inquiry catalogue? This changes the theme and the work.
2. **Same products as Marblebee?** If some products appear on both sites, the same text on two websites can hurt search ranking. The product texts should be made different, or the products should not appear on both.
3. **Photos:** many photos carry the Classybath or MarbleBee watermark. Confirm that is wanted on the new store.
4. **Domain access:** we need access to the domain's DNS (the DNS panel where classybath.com is registered) and to the current hosting.
5. **Old links:** keep the old site until the new one is checked. The redirect list is the part that protects search ranking, so it must be complete.
6. **Blog:** 23 articles to move. Confirm they should be kept.

## 7. Effort (rough)

| Part | Size |
|---|---|
| Scan the old site and build the product list | Small, a few hours of scripting plus checking |
| Import, theme setup, blog and pages | A day or two, mostly theme and content choices |
| Redirects and domain switch | A few hours, plus waiting up to 48 hours for DNS |
| 3D | Depends on who makes the models; adding them to Shopify takes minutes each |

## 8. Deleting discontinued products (separate task)

Luna also asked to delete products that are no longer sold, on both Marblebee (Shopify) and Etsy. Raenia will send the list. This is not started. When the list arrives, the plan is:

- **Do not delete outright at first.** Shopify: set the product to Archived (it can be restored; orders stay). Etsy: Deactivate (can be renewed) or Delete (permanent) once it is confirmed.
- **Shopify:** also add a redirect from each removed page to the nearest category page, so old links and search results do not hit an error page.
- **Our sheet:** mark the rows so the automation does not recreate them (for example by clearing Ready_to_post and noting "discontinued"). Duplicate-product risk: if the Shopify links are cleared in the sheet, the automation creates a new product, so the links must stay.
- **Etsy:** listings published by our workflow have the listing id in the sheet; older ones are matched by SKU, so the list needs product numbers.
- Do a dry run list first (what would be removed, per product), then remove in small batches, and check the sitemap and the sheet afterwards.

## 9. Sources used

- Live site classybath.com (page, sitemap and sample product pages inspected on 2026-11-06).
- Shopify developer documentation, product media (3D models: GLB and USDZ, 500 MB limit): https://shopify.dev/docs/apps/build/online-store/product-media
- Shopify Help Center, URL redirects (bulk CSV import, 100,000 redirect limit, `.html` rule): https://help.shopify.com/en/manual/online-store/menus-and-links/url-redirect
- Shopify Help Center, connect a domain bought elsewhere (A record 23.227.38.65, www CNAME shops.myshopify.com, up to 48 hours): https://help.shopify.com/en/manual/domains/add-a-domain/connecting-domains/connect-domain-manual
- Shopify community and migration guides on keeping SEO during a move: https://community.shopify.com/t/how-can-i-migrate-my-website-to-shopify-without-affecting-seo/205070
- Third-party articles on running two brands (separate stores are the usual choice): https://www.launchtip.com/how-to-make-multiple-shopify-stores-from-one-shopify-account/
- Not verified against Shopify's own pricing page: the monthly price of a second store.
