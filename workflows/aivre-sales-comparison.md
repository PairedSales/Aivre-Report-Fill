> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre sales comparison grid

Fill the Sales Comparison Approach grid for the address the appraiser gives, from the connectMLS (MRED) TSV export they attache, and the comparable part of Prior Sale and Transfer History. Do not work through the rest of the report. Do not change the aivre-report-fill skill; read its "Browser method" section for Aivre browser handling on the non-grid pages.

Aivre's AI features are no longer used, with one exception: the TSV import (Import from CSV/TSV, Analyze CSV/TSV, Apply Fields) is how comps get into the grid. Everything after Apply Fields is checked and entered by hand. Never use Autofill, Commentary Fill Assist, Ava AI Chat or any other Aivre AI tool on the grid. Create map and Create aerial are map tools, not AI, and are still used.

The order is fixed: 1) TSV import, 2) both maps (Sales Comparison Map and Aerial Map), 3) fill the rest of the grid by hand, 4) comparable transfer history.

Never stop to wait for the appraiser's answers. Make the reasonable call, enter it, and list it in the summary.

## Scope
- Do: comp data rows, proximity (via the map), both map exhibits at the bottom of the Sales Comparison page (Sales Comparison Map and Aerial Map), comparable prior sales and transfers.
- Do not: adjustments (leave every adjustment cell blank; clear any value the import entered), comp Quality and Condition ratings (leave Interior/Overall Quality and Condition blank for the appraiser), Comparable Weight, Indicated Value, reconciliation commentary, comp photos. The appraiser does those.
- Done when the only red cells in the grid are the comp photo boxes, the quality and condition ratings, the adjustment cells and the Comparable Weight row, and both maps are saved.

## Before starting
- Use the connected Chrome (see chatgpt-runtime.md). Open the report from https://www.aivre.com/dashboard in a new tab (click the address link; never create a report). The report's own URL (https://www.aivre.com/uad/<id>) also reopens it; use find refs for the left-nav buttons.
- Aivre allows one editor at a time. If the page says "This order is being edited somewhere else", close your tab and ask the appraiser to close their copy of the report, then reopen.
- Read the subject column of the grid first (GLA, beds/baths, floor, Q/C, structure design, garage, PUD). If a subject value is wrong (the appraiser corrects it), fix it in the subject's own section first (e.g. Dwelling - Building 1 for Structure Design), then copy it to the comps.
- Check the engagement letter for comparable requirements and flag any the TSV does not meet. Class Valuation: at least three closed sales and two listings or pendings (or an explanation), MLS photo use disclosed, and a location map showing the subject and every comp including listings.
- Files to upload go in Scratch/<slug>/uploads/, or attach workfile originals.

## 1. TSV import (the one Aivre AI step)
1. Sales Comparison Approach page. Click the arrow on "Add Comparable", choose "Import from CSV/TSV". Do not add comps manually.
2. Attach the TSV (see Uploads in chatgpt-runtime.md) on the dialog's real input[type=file] (there may be two matches; use the one that is not the drop-zone button).
3. Appraiser context box, for example: "Import all rows in this order: <addresses>. All in <project>, same project as subject. LMT column = days on market. Sold Pr = sale price, Closed Date = sale date, Contract Date = contract date, List Price = list price. ASF = finished above grade sq ft. Unit Fl No = floor number. FIN = financing type. Do not enter any adjustments."
4. Analyze CSV/TSV, wait (about 30-60 seconds), read the extraction matrix, then Apply Fields.
5. Check every imported value against the TSV by hand: list price, sale price, contract date, sale date, DOM (= LMT), GLA (ASF), beds, baths, MLS #. Fix anything that differs. Clear any adjustment or Q/C value the import filled. Read input values with JS by row label and column x position; get_page_text does not show input values.

## 2. Both maps (Sales Comparison Map fills Proximity)
Both maps are always created, at the bottom of the Sales Comparison page.
1. Scroll to Sales Comparison Map, click "Create map". Wait until all addresses show as located.
2. Just generate and save the map. Aivre's location map handles balloon placement well now, so do not drag or reposition balloons.
3. Save Map. Proximity to Subject fills in the grid automatically (e.g. "0.21 Miles E"). Never type proximity by hand. Comp latitude and longitude come only from Aivre's own maps, never from county GIS, the MLS or the address autocomplete.
4. Scroll to Aerial Map, click "Create aerial", wait until every address shows located, and Save Aerial, again without modifying it. Saving can take 30+ seconds; wait for the thumbnail. Aivre also adds the aerial as a Sales Comparison Approach exhibit.
5. Check the subject pin on both maps against the subject's confirmed location (the aivre-report-fill aerial). The geocoder has put the subject kilometers away before; if it is wrong, fix the location before saving.

## 3. Fill the rest of the grid by hand
Every remaining cell is entered manually from the TSV, the MLS and the workfile. Right-click a filled cell for Copy right / Copy left / Copy to comp 2-3-4 / Copy to all comps. Set comp 1 (or the subject when its menu appears) and copy across instead of clicking every comp.

The appraiser's data rules for comps:
- Days on market: verify from the connectMLS listing history. Agents often take a listing down and relist it to reset days on market; use the full marketing time.
- Bedrooms and baths from the MLS, as totals including basement rooms (UAD 3.6 does not separate them; "4+1 bsmt" = 5). Verify bedrooms against any comp floorplan in the workfile: a room reachable only through other bedrooms is not a bedroom.
- Above-grade GLA: the assessor size, unless a comp floorplan in the workfile disagrees, in which case the floorplan wins (trace an untotaled floorplan, e.g. with https://pairedsales.github.io/FloorTrace/). Without a floorplan in Lake County, take size and basement finish from the Lake County assessor, but not its basement finish when the MLS shows the basement finished (Lake's finish data is bad).
- Basements: unfinished below-grade area = the township basement total minus the MLS finished basement area. Cook County comp basement sizes usually have to be estimated.
- Attic and other unfinished spaces go on the grid separately from finished area.
- Lot size from the property record card, not the MLS.
- Disclaimers at the start of the Sales Comparison Approach commentary, short and without size sources: which comps' GLA comes from a floorplan instead of the assessor and why, and that basement sizes are estimated where they are.
- A comp with annual association fees is Property in a PUD Yes.
- A subject's own recent sale (e.g. a builder sale) may be gridded as a comp only when the appraiser includes it; it gets no weight.

Typical values for comps in the subject's own condo project:
- Homeowner Exterior Maintenance: No (association covers exterior).
- Attached/Detached: Attached. Property Rights: Fee Simple. Site Owned In Common: Yes.
- Structure Design: same as subject. Oakwood Hills (Elgin) style garden condos are "Apartment", not Rowhouse/Townhouse or Low-rise. Changing Structure Design clears Construction Method, so redo that after.

Typical values for detached houses, including PUD houses:
- Attached/Detached from the MLS (Detached). Property Rights: Fee Simple. Site Owned In Common: No.
- Homeowner Exterior Maintenance: Yes unless the MLS says the association maintains exteriors (lawn and snow service alone does not make it No).
- Property in a PUD: Yes when the MLS shows association fees.
- Structure Design from the MLS style and photos; flag any you judged from photos.

All comps:
- Transfer Terms: "Typically Motivated" for ordinary arm's-length sales (cash or financed, as-is included, builder sales). Use Estate Sale for probate (MLS ASI "Probate"), REO, Short Sale, Court Ordered etc. only when the MLS shows it. The import often leaves this blank or picks "Other (Describe)"; fix those.
- Sales Concessions: No unless the MLS CONC field shows an amount.
- Construction Method: Site Built. Cooling: Yes, Centralized (Central Air) when the MLS shows it. Below-grade areas 0 when no basement.
- Vehicle storage: check the import's attached/detached against the MLS. If the MLS has no garage type, match the project's other units and flag it.
- Interior and Overall Quality and Condition: leave blank for the appraiser. In the summary, note anything in the MLS remarks or photos that bears on them (as-is, estate, long vacant, renovated, new construction).

Dropdown tips: click the chevron at the left of the cell; some open a small popup with a select or checkbox list (type to filter). Press Escape to close popups. The layout shifts after edits and window resizes, so take a fresh screenshot before clicking by coordinate.

## 4. Comparable transfer history
On the Prior Sale and Transfer History page, under Comparable Transfer History, use "Add Comparable" for each comp. Only transfers within one year before the comp's sale date are relevant; take them from the property record card or the connectMLS history (the sale being gridded is not a prior transfer). Read each field's options before choosing; enter No prior sale when there is none in that year. Write a short "Analysis of Prior Sale and Transfer History of Comparable Sales": either the prior transfers found with dates and prices, or that none of the comps transferred in the year before their sale, with the data source.

## When done
Short summary to the appraiser:
- What was filled (including both maps) and that adjustments, Q/C ratings, weights, indicated value and reconciliation were left for them.
- Flags: any value changed from the import (and why), MLS GLA sources (assessor vs floorplan vs estimated) against the subject's measured GLA, days on market corrected for relists, comps on a different floor than the subject, personal property included in a sale, view or amenity values the import pulled from remarks, notes bearing on Q/C ratings, and any engagement-letter comparable requirement the comps do not meet.
- No em dashes in anything written.