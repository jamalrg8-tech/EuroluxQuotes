# Eurolux Quotes — PWA

Quotation app for Eurolux Doors and Windows LLC. It is a single HTML file plus the small files a PWA needs.
It has no build step and loads nothing from the internet, so it also works offline once installed.

```
index.html              the whole app (styles, logic, PDF reader, default logo and cover image)
vendor/                 pdf.js (Mozilla, Apache-2.0), used to read uploaded PDFs
manifest.webmanifest    PWA install details (name, icons, colours)
sw.js                   service worker (offline caching)
icons/                  app icons
vercel.json             cache headers for Vercel
```

## Try it locally
Open `index.html` in Chrome or Edge. Everything works except install/offline, which need http(s).
To test those too, run `npx serve .` (or `python3 -m http.server`) in this folder and open the address it prints.

## Deploy (GitHub → Vercel)
1. Create a GitHub repository, e.g. `eurolux-quotes`, and upload these files to its root.
2. In Vercel: **Add New → Project → Import** the repository. Framework preset: **Other**. No build command, no output directory.
3. Deploy. Every push to `main` redeploys automatically.
4. On a phone, open the Vercel URL and use **Add to Home Screen** (iOS) or **Install app** (Android/Chrome).

When you ship a new version, change `CACHE` at the top of `sw.js` (e.g. `v1.0.1`) so installed copies update.

## Grouping items: Location or Description
Items can be grouped under a free-text **Location or Description** heading (e.g. Ground Floor, Master Bedroom, Villa 12), which prints as a grey bar in the schedule.
Headings are optional: quotes started from scratch have none, while quotes created from a PDF use the headings in the BOQ or drawing (sections or elevation titles). You can rename or clear these on the review screen (↺ restores the document's name) or later in the editor.
Items with no heading print without a bar. Default headings can be set in Settings.
Each item also has an optional **Floor / level** (GF, FF…), printed as "Location: GF" under the description.

## How pricing works
- **Product rates are cost rates.** Sell rate = cost × (1 + markup %). The default markup is set in Settings and can be changed per quote.
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

## New quote from a drawing or BOQ (PDF)
**New quote → Upload drawing or BOQ.** The PDF is read in the browser; nothing is uploaded.
- **BOQ / bill of quantities:** finds the item table by its headings (Ref/Item, Description, Width, Height, Qty…), including sections, marks, sizes, quantities, locations and notes, and follows the table across pages.
- **Drawings:** reads the window/door schedule (Mark, Name, Height, Width) and counts the tags on each elevation ("W-4" with "H=… W=…" under it) to get quantities per elevation.
  Each row of tags on an elevation is treated as a floor level, and you confirm GF/FF/RF per row.
- **Review screen:** match each document description to your products ("Auto" picks by type and size, and your choices are remembered), untick lines you don't want, edit anything, then **Create quote**.
- Flags: very wide or tall tilt & turn sashes, missing sizes, marks not in the schedule, and schedule marks with no tags on the elevations (added unticked).
- Limits: scanned PDFs (images with no text) can't be read. Unusual layouts may need more editing on the review screen.

## Revisions and variation orders
- **Create revision** (in the quote editor): the customer has asked for changes. The current version is kept read-only as "Revised", and a new copy **Q-12345-R1** opens.
  Add, remove or change items, or apply a discount. The editor lists every change and the price difference against the previous revision.
  Revision notes are printed on the terms page, new and changed lines are marked **NEW / R1** in the schedule, and removed lines are listed.
- **+ Variation order** (on an **Approved** quote): extra items after approval. It creates **Q-12345-VO1**, titled "VARIATION ORDER", with a reference to the approved quote and a
  variation summary (original order + approved variations + this variation = revised total). Variation orders can be revised too (Q-12345-VO1-R1).
  The approved quote shows all its variation orders and the running contract value.

## Where data lives (for now)
Everything is saved in the browser on each device (localStorage), so each estimator's quotes stay on their own device until Supabase is added.
Use **Settings → Download backup** regularly, and **Restore from backup** to move data between devices.
Quote numbers count up per device, so two people could issue the same number before Supabase is connected. You can edit the number on any quote.

## Next step: Supabase
All saving and loading goes through the `Store` object in `index.html`. That is the only part that changes.
Planned tables: `profiles` (users), `settings`, `products`, `glass_options`, `extras`, `quotes`, `quote_items`, `quote_extras`.
Supabase Auth adds a login for each user (Steve, Farah, Loureanne, Su Wai, Ed, Fadil), and a database sequence gives shared quote numbers.
