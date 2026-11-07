> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre sales comparison: 2-4 unit

Fill the Sales Comparison Approach grid of a 2-4 unit Aivre (UAD 3.6) report for the address, using what is in its workfile folder. This skill stands on its own. Where it differs from aivre-sales-comparison (single family and condo), this skill wins: comp photos and both maps are done here.

Never stop to wait for the appraiser's answers. Make the reasonable call, enter it, and list it in the summary. No em dashes in anything written.

## Scope
- Do: TSV import, every comp data row, unit rows with estimated sizes, comp photos, Sales Comparison Map and Aerial Map, the size disclaimer at the start of the Sales Comparison reconciliation commentary, comparable transfer history.
- Leave for the appraiser: every adjustment cell, comp Quality and Condition ratings (dwelling, each unit, overall), Comparable Weight, Indicated Value, and their reconciliation analysis after the disclaimer.
- Done when the only red cells left in the grid are Q/C ratings, adjustments, weights and indicated value.

## What is in the workfile
- Comparable Sales TSV (connectMLS export of the gridded sales). Count its rows; do not assume the count.
- Comp Photos folder: one front photo per sales comp, and often the rental comp photos too. File names are HD_<epoch ms>_...; the epoch is the photo upload time, which lands near the listing's list date (convert it and match to each comp's List Date). Some names contain the street. Match the rest by elimination and say so.
- County cards: Realist "Printable View" screenshots (often chrome_*.png), one per comp: year built, lot sq ft, building sq ft, stories, basement type and finish, heat, garage, and Sale History.
- MLS floor plans for some comps (e.g. *_floors_*_with_dim.jpg). A floor plan beats the card for the part of the building it covers.
- The appraiser's subject sketch area calculation page (unit sizes, GLeA, GBA).
- Engagement letter: check comparable requirements (Rocket: every comp, including listings, needs a photo; MLS photos are acceptable).
- If the county cards are missing, tell the appraiser they are missing and carry on with the assessor site (cookcountyassessoril.gov/pin/<14-digit PIN>) as a stopgap.

## Order
1. Read the TSV, the cards, floor plans, subject calc page and the engagement letter.
2. TSV import.
3. Sales Comparison Map, then Aerial Map.
4. Fill the grid by hand.
5. Unit sizes.
6. Comp photos.
7. Size disclaimer.
8. Comparable transfer history.
9. Reload and verify, then summarize.

## Browser basics
- Use the connected Chrome (see chatgpt-runtime.md). Open the report from https://www.aivre.com/dashboard by clicking the address link (a "Get listed in the Aivre Directory" popup may need "Maybe later" first). The report URL https://www.aivre.com/uad/<id>?section=SALES_COMPARISON_APPROACH reopens it.
- If the page says "This order is being edited somewhere else", close the tab and ask the appraiser to close their copy.
- The appraiser may be working in the same tab. If a field of theirs is focused, click blank page area first so their edit saves before you navigate.
- Fit (top bar) shows all comp columns. The layout shifts often; take a fresh screenshot before clicking by coordinate.
- Runtime.evaluate output is blocked when it contains ";" or "&". It is also cut off after about 900 characters: return small chunks.
- One javascript call times out at about 45 seconds. Keep each call to about 20 field sets.
- Never click the first button after a row label or field label: it is "Get AI help" and opens Ava AI Chat with a prefilled prompt. If that happens, clear the prefilled textarea and close the panel with its "Close panel" button (find it by ref). Never use Autofill, Commentary Fill Assist, Ava or "AI Autofill" buttons.

### Grid structure and helper
The grid is a table. Each row is a TR: children[0] is the row label, children[1] the subject, children[2..] the comps. Inside a comp cell, the value is in the left half and the adjustment input is in the right half. Never type into the right half (that is an adjustment). Row labels repeat across Unit Rows 1-4 (and Structure ID appears in Dwelling(s) too), so pass the occurrence index.

Three cell types:
- Dropdown: the cell's first button opens a [role=listbox] with [role=option] buttons (Homeowner Exterior Maintenance, Transfer Terms, Attached/Detached, Property Rights, Property in a Project, Site Owned In Common).
- Popup: the cell's first button opens a div.fixed with class z-[9999] holding inputs, Yes/No radios ([role=radio] aria-label "Yes option"/"No option") and selects (Site Size, Year Built with Year Built Estimated, Construction Method, Heating, Cooling, Vehicle Storage Details, Outdoor Living, Whole Home). Close it by dispatching pointer events on document.body.
- Direct input: Gross Building Finished Area, Finished Area Above Grade, Finished Area Below Grade, Unfinished Area Below Grade.

Inject after every page load:

```js
window.G = {
 sleep: ms=>new Promise(r=>setTimeout(r,ms)),
 rows(){ return [...document.querySelectorAll('tbody tr')].filter(r=>r.children.length>=3); },
 row(l,o=0){ return G.rows().filter(r=>r.children[0].innerText.replace(/\s+/g,' ').trim()===l)[o]; },
 td(l,c,o=0){ const r=G.row(l,o); return r && r.children[c+1]; },  // c = comp number, 0 = subject
 ct(td){ return td.innerText.replace(/\s+/g,' ').replace(/Adjustment|editable|read-only|not-applicable|unavailable|\$/g,'').trim(); },
 click(el){ el.scrollIntoView({block:'center',inline:'center'}); ['pointerdown','mousedown','pointerup','mouseup','click'].forEach(x=>el.dispatchEvent(new PointerEvent(x,{bubbles:true}))); },
 opts(){ return [...document.querySelectorAll('[role=option]')].filter(o=>o.offsetParent); },
 esc(){ document.dispatchEvent(new KeyboardEvent('keydown',{key:'Escape',bubbles:true})); },
 pop(){ return [...document.querySelectorAll('div.fixed')].filter(d=>/z-\[9999\]/.test(d.className)).pop(); },
 async closePop(){ ['pointerdown','mousedown','pointerup','mouseup','click'].forEach(x=>document.body.dispatchEvent(new PointerEvent(x,{bubbles:true}))); await G.sleep(400); },
 async openPop(l,c,o=0){ if(G.pop()) await G.closePop(); const td=G.td(l,c,o); td.scrollIntoView({block:'center',inline:'center'}); await G.sleep(400); G.click(td.querySelector('button')); await G.sleep(900); return G.pop(); },
 async pick(l,c,opt,o=0){ const td=G.td(l,c,o); G.click(td.querySelector('button')); await G.sleep(500); const x=G.opts().find(e=>e.innerText.trim()===opt); if(!x){ const s=G.opts().map(e=>e.innerText.trim()).join('|'); G.esc(); return 'NOTFOUND '+s; } x.click(); await G.sleep(600); G.esc(); await G.sleep(200); return G.ct(G.td(l,c,o)); },
 async popSelect(i,opt){ const b=[...G.pop().querySelectorAll('button')][i]; G.click(b); await G.sleep(700); const x=G.opts().find(e=>e.innerText.trim()===opt); if(!x){ G.esc(); return 'NOTFOUND'; } x.click(); await G.sleep(600); G.esc(); await G.sleep(300); return 'ok'; },
 async setEl(el,v){ el.focus(); Object.getOwnPropertyDescriptor(HTMLInputElement.prototype,'value').set.call(el,v); el.dispatchEvent(new Event('input',{bubbles:true})); el.dispatchEvent(new Event('change',{bubbles:true})); await G.sleep(200); el.blur(); el.dispatchEvent(new FocusEvent('focusout',{bubbles:true})); await G.sleep(450); },
 valInp(l,c,o=0){ const td=G.td(l,c,o); const R=td.getBoundingClientRect(); return [...td.querySelectorAll('input')].filter(i=>i.type==='text').find(i=>i.getBoundingClientRect().x < R.x+R.width*0.5); },
 async setv(l,c,v,o=0){ const el=G.valInp(l,c,o); el.scrollIntoView({block:'center',inline:'center'}); await G.sleep(150); await G.setEl(el,v); return G.valInp(l,c,o).value; }
};
```

- Popup inputs: `const p=await G.openPop('Year Built',c); await G.setEl(p.querySelector('input'),'1901');` then set the Estimated radio and `await G.closePop()`. Read back with G.ct.
- Popup selects: look at the popup's buttons first (`[...G.pop().querySelectorAll('button')].map(b=>b.innerText)`) and pass the index of the one to change. Cooling: click the "Yes option" radio, then popSelect(0,'Centralized'). Vehicle Storage: button 2 is Parking Space Assignment Type.
- A popup sometimes does not open for a column scrolled off screen. Scroll the cell into view, wait, and retry once.
- To remove an item in a popup (a wrong Whole Home amenity), click its "Remove this ..." button.
- Comp structure (add a dwelling, outbuilding or unit to one comp) is in the comp's header gear, "Manage structures".

## 1. TSV import
1. Sales Comparison Approach page, arrow on "Add Comparable", "Import from CSV/TSV". Attach the TSV to the dialog's real file input (see Uploads in chatgpt-runtime.md).
2. Appraiser context, for example: "Import all N rows in this order: <addresses>. These are 2-4 unit buildings. LMT column = days on market. Sold Pr = sale price, Closed Date = sale date, Contract Date = contract date, List Price = list price. BR1-BR4 = bedrooms per unit, Full/Half Baths-Unit 1-4 = baths per unit, Flr # - Un = unit floor. FIN = financing type. CONC = sales concession amount. Do not enter any adjustments and do not enter quality or condition ratings."
3. Analyze CSV/TSV (about 60-90 seconds), then Apply Fields.
4. Check every imported value against the TSV: prices, dates, days on market, MLS #, per-unit bedrooms and baths. Clear anything typed into an adjustment.

The import's habitual mistakes:
- Transfer Terms blank or "Other (Describe)".
- Year Built set to "Yes" (the Estimated flag) with no year.
- Site Size from the MLS lot field, which can hold the building size.
- Property in a Project = Yes with the subdivision or neighborhood as the project name.
- Parking "Assigned" from the MLS parking details.
- Whole Home amenities guessed from nothing (e.g. Smart Home System).
- A coach house entered as an Outbuilding even when it is one of the units.

## 2. Maps
Both maps go at the bottom of the Sales Comparison page:
- Sales Comparison Map: "Create map", wait until every address shows located, Save Map. Proximity to Subject fills itself. Never type proximity.
- Aerial Map: "Create aerial", Save Aerial. Aivre also adds it as a Sales Comparison exhibit.
- Do not drag or reposition balloons. Check the subject pin is in the right place before saving.

## 3. Grid rows
- Homeowner Exterior Maintenance: Yes. Attached/Detached: Detached (from the MLS). Property Rights: Fee Simple. Property in a PUD and in a Project: No. Site Owned In Common: No.
- Transfer Terms: Typically Motivated for ordinary sales, cash or financed. Use Estate, REO, Short Sale and the like only when the MLS shows it.
- Sales Concessions: from CONC.
- Site Size: the county card's lot sq ft.
- Year Built: the county card. Year Built Estimated No when the card gives a year.
- Gross Building Finished Area: the county card's building sq ft, unless a floor plan in the workfile shows the card is wrong for part of the building.
- Construction Method: Site Built.
- Heating: from the MLS. If the MLS is blank or "None", use the card (e.g. "Warm Air" = Forced Warm Air). Leave the fuel blank when no source gives it.
- Cooling: Yes, Centralized when the MLS unit features or the card show central air.
- Unit rows: bedrooms and baths per unit from the MLS (already imported). Verify bedrooms against any floor plan.
- Outdoor Living and Whole Home: from the MLS features and remarks only. Remove import guesses with no support.
- Vehicle Storage: type and count from the MLS. Parking that comes with the building is Owned, not Assigned.
- Leave every Q/C rating blank.

## 4. Unit sizes (always estimate)
Cook County size data is poor, and agents rarely measure or provide sketches, so every comp unit gets a reasonable estimate. Leave no unit size blank. Use everything available:
- The county card building sq ft and stories.
- MLS unit floor numbers (1, 2, 3, G or L = garden/lower, C = coach house), bedroom, bath and room counts per unit, unit sizes when the MLS gives them, basement and duplex remarks.
- Floor plans in the workfile. When a floor plan has room dimensions but no total, scale it in pixels from a known room width and measure the outer walls.
- The City of Chicago building footprints dataset (Socrata syp8-uezg: shape_area, stories, year_built), queried from the browser with fetch.

Method:
1. Decide how many levels the card's building sq ft covers. The Cook assessor excludes basements "except apartment", so a building with a basement or duplex-down unit may include that level. Test the options against the footprint and the lot: a per-floor area larger than the footprint is impossible.
2. Per-level gross area = card building sq ft / levels.
3. Net unit area is about 92% of gross per level (common stairs and halls). The subject's own calc page shows the actual ratio; use it.
4. Split each level among its units (front and rear, or side by side). Split unequal units by room count. Duplex units get one level above grade plus one below.
5. Round to the nearest 10 sq ft. A traced floor plan keeps its traced figure.
6. Garden or lower-level units (floor G or L) are below grade: Finished Area Above Grade 0, Finished Area Below Grade = the estimate. Units with no below-grade space get 0 below grade. Unfinished Below Grade is 0 unless a source shows unfinished space.
7. When the MLS and the card disagree on a basement (none vs full, finished vs unfinished), follow the MLS and the unit layout, and list the conflict.

Subject check: a subject garden unit (first-floor unit partly below grade) must show as below grade. On its Unit Interior page set Grade Level Type to Partially Below Grade (or Fully Below Grade). Then fill Below Grade Access, Below Grade Finish Compared to Above, Below Grade Exterior Access, Finished Above Grade 0 and Finished Below Grade = the unit area. List the access choices for the appraiser.

## 5. Comp photos
- Compress each matched photo to about 1600 px on the long edge and put it in Scratch/<slug>/uploads/<comp address>.jpg.
- Grid photo cells: in the Property Address row, the file inputs are in children[1] (subject) and children[2..] (comps). Give each comp's input a unique aria-label by script, then attach the photo to that input (see Uploads in chatgpt-runtime.md).
- Zoom on the photo row afterwards to confirm each photo sits in the right column.

## 6. Size disclaimer
The Sales Comparison Approach Reconciliation commentary always starts with:

"Building size data in Cook County is not reliable, and it is not common practice for agents to measure, verify or provide third-party sketches. In many cases, reasonable estimates are the best available data source for comparable building and unit sizes. Sizes in the grid may vary from other appraisals in the area, but the data in the grid is considered reasonably accurate."

This textarea does not save from a JS value setter. Click into it, select all, type the text with the keyboard, then click blank page area. After a reload, JS may read it as empty while it is saved: check it with a screenshot or zoom.

## 7. Comparable transfer history
- Source: the county card Sale History, cross-checked with the Cook County Clerk recordings by PIN. Open https://crs.cookcountyclerkil.gov/Search/ResultByPin?id1=<PIN> in a tab. From that tab, fetch the other PINs by script (same origin) and parse the rows: doc number, recorded and executed dates, doc type.
- Only transfers within one year before the comp's sale date count. The deed for the gridded sale is not a prior transfer.
- On Prior Sale and Transfer History, each comp already has a tab. Set Prior Sales or Transfers Exist (No when none in that year) and Data Source "Deed" (the + after Data Source, aria-label "Add option").
- Analysis, short, e.g.: "None of the comparable sales transferred in the year before their sale dates, per the Cook County Clerk recorded document search." Or list each one found with date and price.
- Note transfers just outside the year (flips, quit claims) in the summary.

## 8. Verify
Reload the report and read back with the helper: every row that was changed, every unit size, Year Built and Gross Building Finished Area. Check the left-nav red counts: Prior Sale and Transfer History should be clear. Leave the report tab open for the appraiser.

## Summary to the appraiser
Short, no em dashes:
- What was filled and what was left for them.
- The unit size estimate table (comp, each unit above and below grade, and how it was derived).
- Values changed from the import, and why.
- Card vs MLS conflicts: basement, garage, heat, year built.
- Card building sizes that look wrong, or that count space on a different basis from the subject's GBA (e.g. includes a basement or garden level).
- Photos matched by elimination.
- Sale conditions bearing on adjustments or Q/C: concessions, owner financing offered, vacant units, off-market or private exposure, prior flips, renovation dates and scope.
- Engagement-letter comp requirements not met.