# Eurolux Quotes — PWA

Quotation app for Eurolux Doors and Windows LLC. It is a single HTML file plus the small files a PWA needs.
It has no build step and loads nothing from the internet, so it also works offline once installed.
The PDF reader and the Excel reader are built into `euroluxquotes.html`, so uploading `euroluxquotes.html` alone is enough for every feature (the other files add install/offline support).

```
euroluxquotes.html              the whole app (styles, logic, PDF and Excel readers, default logo and cover image)
manifest.webmanifest    PWA install details (name, icons, colours)
sw.js                   service worker (offline caching)
*.png                   app icons (icon-192, icon-512, maskable-512, apple-touch-icon)
pdfjs-LICENSE.txt       licence for pdf.js (Mozilla, Apache-2.0), which is built into euroluxquotes.html to read PDFs
sheetjs-LICENSE.txt     licence for SheetJS (Apache-2.0), which is built into euroluxquotes.html to read Excel BOQs
vercel.json             cache headers for Vercel
```

## Try it locally
Open `euroluxquotes.html` in Chrome or Edge. Everything works except install/offline, which need http(s).
To test those too, run `npx serve .` (or `python3 -m http.server`) in this folder and open the address it prints.

## Deploy (GitHub → Vercel)
1. Create a GitHub repository, e.g. `eurolux-quotes`, and upload these files to its root (there are no folders, so **Add file → Upload files** works directly).
2. In Vercel: **Add New → Project → Import** the repository. Framework preset: **Other**. No build command, no output directory.
3. Deploy. Every push to `main` redeploys automatically.

The app file is `euroluxquotes.html`. `vercel.json` sends the site address (`/`, and old `/index.html` links) to it, so the URL stays https://euroluxquotes.vercel.app/. There must be **no `index.html`** in the repository: Vercel serves a real file before it applies these redirects, so an old `index.html` would keep showing the old app. If you host the app somewhere else, keep equivalent redirects or open `/euroluxquotes.html` directly.
4. On a phone, open the Vercel URL and use **Add to Home Screen** (iOS) or **Install app** (Android/Chrome).

When you ship a new version, change `CACHE` at the top of `sw.js` (e.g. `v1.4.1`) so installed copies update.

## Grouping items: Location or Description
Items can be grouped under a free-text **Location or Description** heading (e.g. Ground Floor, Master Bedroom, Villa 12), which prints as a grey bar in the schedule.
Headings are optional: quotes started from scratch have none, while quotes created from a PDF use the headings in the BOQ or drawing (sections or elevation titles). You can rename or clear these on the review screen (↺ restores the document's name) or later in the editor.
Items with no heading print without a bar. Default headings can be set in Settings.
Each item also has an optional **Floor / level** (GF, FF…), printed as "Location: GF" under the description.

## How pricing works
- **Product rates are cost rates**, loaded from *2026 Doors & Windows Rates* (Eurolux Rates sheet; Eurolux local systems use the "other clients" column). Sell rate = cost × (1 + markup %) × (1 + category uplift %). The default markup is set in Settings and can be changed per quote. Existing installations get the new rates once when they open version 1.4.0.
- **Pricing units:** per m² (width × height), per linear metre (width = length, e.g. balustrades), or each (e.g. doors).
- **Width bands** (optional, per product): a different cost rate when the width falls inside a range.
  Example: AWS 65 Tilt & Turn is 3,800 standard, 4,000 up to 600 mm wide, and 3,500 from 1,200 mm.
- **Minimum chargeable m² / lm** (optional, per product).
- **Glass upgrades:** extra cost per m², chosen on each item (or applied to all items at once). Markup is added on top.
- **Manual rate:** any line can override the calculated rate per item. It is flagged "MANUAL RATE" in the editor, but not on the printed quote.
- **Additional charges:** installation (per m² of the quote, or lump sum), mobile crane (per day), removals (per item), delivery. These are entered at selling price, with no markup.
- **Discount:** % or fixed AED, on items only or on items + additional charges.
- **VAT:** 5% by default (Settings).
- An internal panel shows cost, markup, profit and margin. It is never printed.

## New quote from drawings or a BOQ (PDF or Excel)
**New quote → Upload drawings or BOQ.** Choose one file or several together (e.g. the Excel BOQ plus the architectural PDF). Files are read in the browser; nothing is uploaded.
- **BOQ in Excel (.xlsx/.xls/.csv) or PDF:** finds the item table by its headings (Item/Ref/Section No., Description, Location, Width/Length, Height or "Width x Height", Qty, Unit), including section headings, rooms, notes ("Door: 950 mm") and sizes written in the description ("Type (GD01) - (160 cm x 340 cm)", "ALW 002 (3000 x 3100)"), in mm, cm or m. Tables carry on across pages. Lengths in metres become one line of that length. Equipment lists and other unrelated tables are skipped.
- **Window/door type schedules** (each type drawn with a WINDOW NO. / LOCATION / TOTAL NO. block, dimensions in metres or mm): size, location, quantity and panel sizes per type. Sills below windows are left out of the height.
- **BOQ + drawings together:** BOQ lines are matched to the drawings by mark (GD01 = GD-01, CW7a = CW-07A); locations and panel sizes are added, differences in size or quantity are flagged, and types on the drawings that are missing from the BOQ are added unticked.
- **Elevations with tags** ("W-4" with "H=… W=…" under it): quantities per elevation, each row of tags treated as a floor level.
- **Measure on drawing** (drawings with no marks or schedule, and scanned drawings): drag a box around each opening; width and height are offered from the dimension text next to it, or from the drawing scale. Pick product, ref, qty and location, then Add.
- **Review screen:** each description is matched to a product (by wording, size, mark prefix such as CW/GD/AD, and the BOQ section text); your changes are remembered, and so are heading renames. Lines with no matching product (louvres, shutters, gates, "as per approved visual") are left unticked.
- **Descriptions on the quote** follow the Excel template style: product name plus configuration ("(S+S)"), "(FROSTED)", "(Aluminum Sandwich Panel)", door/fixed panel sizes under the line, and the document's own wording for shower glass and balustrades.
- Limits: scanned PDFs have no text, so only Measure on drawing works on them. Which opening type a curtain-wall or "ALW" line is often isn't stated in the BOQ: check the product on each line.

## Categories and category uplift
- Categories: Exterior Doors and Windows, Insect Screens, Smart Film (SSF), Interior Doors and Windows, Balustrades, Glass Stairs, Others. Add, rename or delete them in **Settings → Categories**.
- Each product has a category (Products → Edit); an item can be moved to another category in the item form.
- **Group by** (quote editor, Items): By location (Location or Description headings) or By category. The printed quote uses the same grouping.
- **Uplift %** per category: a default in Settings, changeable on each quote in the Pricing panel. Sell rate = cost × (1 + markup %) × (1 + uplift %). Negative values reduce. Not shown on the printed quote.

## Revisions and variation orders
- **Create revision** (in the quote editor): the customer has asked for changes. The current version is kept read-only as "Revised", and a new copy **Q-12345-R1** opens.
  Add, remove or change items, or apply a discount. The editor lists every change and the price difference against the previous revision.
  Revision notes are printed on the terms page, new and changed lines are marked **NEW / R1** in the schedule, and removed lines are listed.
- **+ Variation order** (on an **Approved** quote): extra items after approval. It creates **Q-12345-VO1**, titled "VARIATION ORDER", with a reference to the approved quote and a
  variation summary (original order + approved variations + this variation = revised total). Variation orders can be revised too (Q-12345-VO1-R1).
  The approved quote shows all its variation orders and the running contract value.

## User manual
**Help** in the top bar opens the user manual, which is built into the app (everyone can see it, even offline; **Print manual** gives a PDF). To use an external manual instead, put its link in **Settings → User manual link**.

## Line comments
Every item can have a team comment (💬 on the line, or the Comment box in the item form). **Show comments** above the item list hides or shows them. They print only when **Line comments** is ticked on the Preview & PDF screen.

## Where data lives (for now)
Everything is saved in the browser on each device (localStorage), so each estimator's quotes stay on their own device until Supabase is added.
Use **Settings → Download backup** regularly, and **Restore from backup** to move data between devices.
Quote numbers count up per device, so two people could issue the same number before Supabase is connected. You can edit the number on any quote.

## Next step: Supabase
All saving and loading goes through the `Store` object in `euroluxquotes.html`. That is the only part that changes.
Planned tables: `profiles` (users), `settings`, `products`, `glass_options`, `extras`, `quotes`, `quote_items`, `quote_extras`.
Supabase Auth adds a login for each user (Steve, Farah, Loureanne, Su Wai, Ed, Fadil), and a database sequence gives shared quote numbers.
