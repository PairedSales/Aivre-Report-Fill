> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre rental comps (2-4 unit)

Enters individual-unit rental comparables into the Rental Information section of the appraiser's Aivre URAR (UAD 3.6) report and compares every comp to every subject unit. Use it with the aivre-report-fill core (Browser method, Rules) and the aivre-fill-2-4-unit skill already loaded or loaded now. Never stop to wait for the appraiser: make the call, enter it, flag it in the summary.

No Aivre AI features (Autofill, Commentary Fill Assist, "Get AI help" sparkle buttons, Ava). Create map and Create aerial are map tools and are used.

## Where things are
- Workfile folder for the address:
  - the rental listings PDF (connectMLS Residential Rental detail sheets, e.g. "Rental Listing Comps.pdf"), one listing per comp
  - a comp photos folder (e.g. "Comp Photos"): one front photo per comp, file names do not say which comp
  - Rents.png or a rent roll and the sketch calculation page (subject units, for context only; the subject side is already filled by aivre-fill-2-4-unit)
- Aivre: Rental Information > Comparable Rental Properties (the comp list), Comparable Rental Analysis (one grid per subject unit, "Summary - Building 1 - Unit n" in the nav), Rental Comparable Map, Aerial Map, Rental Analysis Commentary.
- The subject's parking: the report's Vehicle Storage section (type and Dedicated Parking Spaces).

## Step 1. Read the listings
Extract with `pdftotext -layout` and grep, per listing: address and unit, MLS #, Type (Condo / Flat / Low Rise / blank), Lot Dimensions ("COMMON" means a condominium unit), Unit Floor Lvl, Beds, Baths, Square Feet and Square Feet Source, Rented Price (Monthly), Rented Date, Available As Of, Monthly Rent Incl (utilities), Parking (# spaces, Is Parking Included in Price, parking fee), Laundry, Furnished, and the remarks ("top floor", rehab year, move-in date, parking fee).

Match each comp photo to its listing by rendering the top of each listing's first page (pdftoppm, crop the photo area) next to a contact sheet of the photos. Copy the photos, compressed (long edge about 1600 px), to Scratch/<slug>/uploads/, named by comp address.

## Step 2. Comparable Rental Properties (the comp list)
Aivre's Comparable Rental Analysis can only select comps from this list, so every unit comp is entered here first. Click "Add Comparable Unit" once per comp. Per comp column:
- Property Address: click the cell, type the street in Street Address, wait about 2 seconds and pick the Chicago autocomplete suggestion (it fills City, State and ZIP), then type the Unit # in its own field. Check State is filled: with State blank the address never geocodes (Geospatial stays "Select"). Fix it in the popover's State listbox (click its search box, type IL, click IL).
- Geospatial Information: fills itself once the address is complete. Confirm it on the Rental Comparable Map (Step 4); never take it from county GIS or the MLS.
- Photo: the front photo. Find the file inputs in the Comparable Rental Properties grid photo cells and attach it to that comp's input with DOM.setFileInputFiles.
- Structure Type: Dwelling (for a unit in a building; Outbuilding only for a coach house unit).
- Lease Start Date (MM/YYYY): the month the lease began = the later of Rented Date and Available As Of. When the remarks give a different move-in date, use the MLS fields and flag it.
- Actual Rent: Rented Price (Monthly), never the list price.
- Data Source and ID: click "Add Data Source" ONCE. That single click adds "Data Source 1" and opens its popover. Do not click the cell again before filling it: each extra click adds another, empty data source (an earlier run left an empty Data Source 1 on every comp that the appraiser had to clear). In the open popover, open the Data Source dropdown, click its search box, type MLS, pick MLS, then type the MLS # in Data Source ID. Close the popover by clicking blank page space. Each comp must show exactly one data source (the "Add Data Source" counter reads 1); delete any empty extra with its trash icon (no confirmation dialog).

## Step 3. Comparable Rental Analysis
- Every comp goes in every subject unit's grid. Use each grid's "Select Comparable Unit from Above"; its menu lists only comps not yet in that grid. Helper (after the core's `A` helper):
```js
window.addComp = async (g,n)=>{ const b=[...document.querySelectorAll('button')].filter(b=>b.innerText.trim()==='Select Comparable Unit from Above')[g]; b.scrollIntoView({block:'center'}); b.click(); await A.sleep(700); const o=[...document.querySelectorAll('*')].filter(e=>e.offsetParent && e.children.length<3 && e.innerText && e.innerText.trim().startsWith('Comparable '+n+':')).pop(); if(!o){A.esc(); return 'no option '+n;} o.click(); await A.sleep(1500); return 'added '+n+' to grid '+g; };
```
  Run it for every grid and comp with a pause between calls, then count "N Comparables" next to each button and rerun any that failed. Aivre keeps the columns in comp-number order.
- Shared vs per grid: a comp's data fields (PUD, Project, Floor Number, Site Owned in Common, Bedrooms, Baths, Finished Area) are shared, so enter them once in any grid and they show in every grid. Rows added from the Rows menu are shared too. The subject unit's Vehicle Storage and each comp's Vehicle Storage comparison are per grid: fill them in all four grids.
- Rows menu: add Floor Number (relevant for stacked units).
- Per comp, fill every row:
  - Proximity to Subject: automatic.
  - Property in a PUD: No.
  - Property in a Project and Site Owned in Common: Yes when the MLS Type is Condo or Lot Dimensions is "COMMON" (condominium unit); No otherwise (e.g. Low Rise with a lot size). Flag the ones decided only from "COMMON".
  - Floor Number: Unit Floor Lvl. When blank, a "top floor" remark plus the unit number supports it; enter it and flag it.
  - Bedrooms: from the MLS (check the room list for rooms that are not real bedrooms).
  - Baths: popover with Full Baths and Half Baths inputs.
  - Finished Area: popover input in sq ft = MLS Square Feet when it is above 0. Leave it blank when it is 0 or not reported.
  - Vehicle Storage, subject column (each unit grid): popover, Vehicle Storage Type from the report's Vehicle Storage section (e.g. Open Lot) and Parking Spaces for that unit. When the spaces cannot be split evenly among the units (e.g. 2 pad spaces for 4 units), enter 1 per unit and flag it.
  - Vehicle Storage, comp columns (each unit grid): the cell is only an Inferior / Similar / Superior comparison to the subject unit. Similar = parking included like the subject unit; Superior = more included spaces or a garage; Inferior = no parking or parking at an extra fee. Helper to set it by grid and comp index:
```js
window.sleep=ms=>new Promise(r=>setTimeout(r,ms));
window.rowBtns=(label,gi)=>{ const labs=[...document.querySelectorAll('*')].filter(e=>e.offsetParent && e.children.length===0 && e.innerText && e.innerText.trim()===label && e.getBoundingClientRect().x>300).sort((a,b)=>a.getBoundingClientRect().top-b.getBoundingClientRect().top); const lab=labs[gi]; lab.scrollIntoView({block:'center'}); const r=lab.getBoundingClientRect(); return [...document.querySelectorAll('button[aria-label=Options]')].filter(e=>{const b=e.getBoundingClientRect(); return e.offsetParent && Math.abs(b.top+b.height/2-(r.top+r.height/2))<10;}).sort((a,b)=>a.getBoundingClientRect().x-b.getBoundingClientRect().x); };
window.rate=async(label,gi,ci,val)=>{ const b=rowBtns(label,gi)[ci]; b.click(); await sleep(600); const by=b.getBoundingClientRect().bottom; const o=[...document.querySelectorAll('button')].filter(e=>e.offsetParent && e.innerText && e.innerText.trim()===val && e.getBoundingClientRect().top>=by-2 && Math.abs(e.getBoundingClientRect().x-b.getBoundingClientRect().x)<60)[0]; if(!o){ A.esc(); return 'NO '+val; } o.click(); await sleep(900); return b.innerText.trim(); };
```
  Then e.g. `await rate('Vehicle Storage', 0, 4, 'Inferior')` and read back with `rowBtns('Vehicle Storage', g).map(b=>b.innerText)`.
  - Leave for the appraiser: Adjusted Rent, Comparable Weight, Opinion of Market Rent, Opinion of Market Income and the Income Approach (GRM).
- Popovers: Yes/No lists open under the cell (Yes about 19 px, No about 39 px below it). Commit each entry by clicking blank page space, then zoom on the grid to read it back. Clicking an already-open dropdown closes it.

## Step 4. Maps
- Rental Comparable Map > Create map. Check every comp is located and each pin sits on its block (compare the balloon coordinates with the grid's Geospatial values). Save Map and wait (30+ seconds) until the modal closes and the thumbnail appears.
- Aerial Map > Create aerial (always, it is the second map). Check the pins the same way, Save Aerial and wait for the thumbnail.

## Step 5. Rental Analysis Commentary
Start the commentary with the size disclaimer, short and factual, no em dashes, e.g.: "MLS rental listings commonly do not show accurate unit sizes. Listed sizes are agent estimates, even when floorplan sketches are uploaded to the listing. All available sources, sometimes including aerial measurements, were used to verify that comparable sizes are reasonable, and the appraiser's estimates are considered more reliable than the listed sizes. As a result, sizes in this report may vary from other appraisals and from the source data." the appraiser adds their rent analysis after it. Reload the page and read the text back to confirm it saved.

## Final check
Before the summary, zoom on the Comparable Rental Properties grid: no comp's Data Source and ID cell may be red (red means an empty extra data source), and every comp shows its address with unit, photo, geospatial, Dwelling, lease month, rent and "MLS #".

## When done
Send the appraiser a short summary:
1. Comps entered: address and unit, rent, lease month, MLS #.
2. Judgment calls: lease start month where the remarks differ, Project/Site Owned in Common decided from "COMMON", floor numbers from remarks, subject parking spaces per unit, the vehicle storage comparisons, sizes that look wrong.
3. Still red and why: missing comp sizes, Adjusted Rent, Comparable Weight, Opinion of Market Rent.
4. Each comp's parking (spaces, included or extra fee) and utilities included, for their adjustments.

Leave the report tab open for their review.