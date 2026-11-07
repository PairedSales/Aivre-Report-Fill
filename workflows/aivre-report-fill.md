> ChatGPT: this workflow was converted from a skill written for another assistant. Read `chatgpt-runtime.md` for what its tool names mean here, and `../AGENTS.md` for the rules that always apply.

# Aivre report fill (core)

Fill the Aivre report for the address the appraiser gives as completely as the documents and photos support, up to (not including) the Sales Comparison Approach, then the Reconciliation defaults (not for 2-4 unit reports) and the Footer. If they give no address, ask for it. The appraiser's command may include extra instructions (condition, sketch, market list, project file, a single section such as the contract). They override anything below. When they ask for one section only, do only that section.

Aivre's AI features are no longer available. Every field is entered by hand from the documents and ChatGPT's own review of the photos. Never use Aivre Autofill, Commentary Fill Assist, Ava AI Chat, "Get AI help for <field>", Import from Sketch, Autofill from Photo, or any other Aivre AI fill, suggest or extract button. The only Aivre AI feature still used anywhere is the TSV import in the Sales Comparison grid, which belongs to the aivre-sales-comparison skill, not this one. Create aerial and the location maps are map tools, not AI, and are still used.

Never stop to wait for the appraiser's answers or confirmations. They review everything at the end: make the reasonable call, enter it, and list it under "Judgment calls to confirm" in the summary. This includes engagement-letter "stop and contact the AMC" items: keep filling and put them first in the summary.

## Property type modules
This skill is the shared core: the workflow and every rule that applies to all reports. Rules that apply to only one kind of report live in their own skills:
- **aivre-fill-single-family:** a house on its own fee simple lot, detached or attached (a PUD house, a fee simple townhouse).
- **aivre-fill-condo:** a unit in a condominium or cooperative project.
- **aivre-fill-2-4-unit:** a property with two to four dwelling units on its own lot (2-flat to 4-flat, duplex, house plus coach house unit).
- **aivre-fill-hybrid:** a hybrid assignment, where a third-party property data collector inspected and the appraiser did not. Hybrid is how the property was inspected, not what it is, so it stacks on top of the property-type skill.

At the start of Step 1, as soon as the order and the workfile file list show the type, pick the modules and read their files (workflows/<name>.md) before going further (skip any already loaded in this conversation):
- Hybrid when the order or engagement letter says hybrid (e.g. Class Valuation "3.6 SFR Hyb", Product-SFR-Hybrid-*.pdf) or the workfile has a Property Data Report ("Data collection report.pdf").
- Condo when the order, engagement letter, legal description ("Unit ... in ... Condominium") or public record shows a condominium or cooperative unit, or the workfile has project.csv.
- 2-4 unit when the order or ErrorLog shows a multifamily order (order type "Multi Family 2-4 Units", "3.6 Multifamily", property type "Multi Family", Form 1025), the sketch shows more than one unit or a Gross Building Area / Gross Leasable Area page, the public record shows a 2-4 unit class (Cook County class 2-11), or the workfile has Rents.png or a rent roll.
- Single-family only for one house on its own lot: an order or record that says single family, one dwelling unit on the sketch, and none of the signs above.

Load exactly one property-type skill, plus aivre-fill-hybrid for a hybrid. When the signs conflict (e.g. a single-family order for a building the sketch shows with two units), go by the sketch and the photos, keep going, and put the conflict under Check first (a wrong property type on the order is a stop-and-contact item for some AMCs).

The modules use this skill's section names. In each section apply the core lines, then the loaded modules' lines for that section. Where they differ, the appraiser's command wins, then aivre-fill-hybrid, then the property-type skill, then this core.

When a rule changes, put it where it applies: here if it applies to every report, otherwise in that one module. Never copy a core rule into a module.

## Where things are
- Workfile: the address's workfile folder, named for the address with dashes (the slug, e.g. 123-Main-St-Anytown-IL-60000).
  - Location: Workfiles/<slug>/ in the Aivre-Report-Fill folder. If a file the task needs is not there, look in Downloads (AGENTS.md section 2).
  - The folder holds:
    - order documents (engagement letter or AMC order page, lender requirements); ErrorLog.txt from the nightly fetch (county, PIN, what was captured, and notes such as county conflicts and street spelling)
    - the public record (tax page), plat map, zoning and flood map captures, and sometimes a survey or seller update list. Will County has no tax page from the nightly fetch; the appraiser adds the Will County CCAO parcel page (property-card-<slug>.png: owner, subdivision, legal description, last sale and document number).
    - the county legal description as a photo (legal-description-<slug>.png; files are named document type first, then the address). Lake County and Will County orders have none.
    - the inspection photos and any cropped detail photos, once the appraiser has added them
    - sometimes market charts, a CropGuide.png, a sales/listings list image or connectMLS TSV, a floorplan sketch and a template sketch
    - the files particular to a condo, 2-4 unit or hybrid assignment, listed in those skills
    - sometimes the subject's MLS sheet (it may be a screenshot named chrome_*.png). See Rules for which MLS data may be used.
  - Treat every file there as read only. Work on copies in the scratchpad. New files ChatGPT makes for the workfile (compressed contract, Sketch.png) may be written there.
- Email: only when the workfile lacks the borrower, fee, loan number or lender/client block, search Gmail for the address and the AMC order number. AMC order emails often carry the borrower name, fee, due date, loan number and lender/client block that the PDFs leave out. Class Valuation sends "New Assignment Notice" (order ID, fee, loan number, client, Illinois registration) and "Status Update for Order <ID>" (PDC name, due date, photo requirements). AMC street addresses are not in the order emails; take them from an AMC staff email signature. Class Valuation: 2600 Bellingham Dr #100, Troy, MI 48083.
- Report: https://www.aivre.com/dashboard in the appraiser's connected Chrome (see chatgpt-runtime.md). A blank URAR (UAD 3.6) report for this address usually already exists. Open it; never create a duplicate. Dismiss any "Aivre Directory" popup with "Maybe later". Only when no report exists and the appraiser asks for a new one, create it with Create new > Uniform Residential Appraisal Report > Create report > Start blank, entering the address. If the appraiser is not logged in (including a two-factor code prompt), stop and tell them; never enter or work around the code. The report's own URL (https://www.aivre.com/uad/<id>) can be reopened directly; it lands on Assignment Information.

## Step 1. Read everything first
Read every file in the workfile, including images and PDFs, and the AMC order emails when needed. When the workfile has the full engagement letter PDF, read all of it: the engagement-letter PNG from the nightly fetch is page 1 only, and lender requirements and stop items are on the later pages. Note:
- borrower, lender/client and AMC (with address, state registration number and expiration), loan number or client number, AMC order number
- fees, assignment type and purpose, inspection date and time
- legal description, PIN, zoning, lot size, flood zone and FIRM panel, easements, private road labels ("PVT")
- owners of record, subdivision, last transfer (date, amount, document number)
- the engagement letter's required elements and its "stop and contact" list (Class: new or proposed construction, subject listed for sale on a refinance, Q6/C5/C6, private road, GLA much different from public record, and others). Never stop for these; list any that apply first in the summary.
- AMC access/order instructions (e.g. Solidifi "verify the sales price and financial assistance from the purchase agreement")
- anything else stated

Cross-check the documents against each other as you read and keep a conflicts list for the summary. Conflicts seen before:
- county: orders have said Kane or Du Page when the property was in Cook or Will. Use the record and tell the appraiser.
- coordinates: the report's pre-filled latitude/longitude and the first aerial pin can be kilometers off; compare with the plat, zoning and flood captures.
- road: a document said public while the county tax map and property card said "(PVT)".
- GLA: builder/MLS square footage vs the measured ANSI area.
- market graphs: the Infosparks filter footnote under each graph names the property class; graphs captured for the wrong class (e.g. Single Family for a 2-4 unit order) are not used.
- The modules list the conflicts particular to them.

If a file is added to the workfile while you are working, read it too.

## Step 2. Review the photos and build a photo log
This replaces Aivre Autofill. Do it once per report; for a single-section request, review only the photos that section needs.

1. View every inspection photo on contact sheets (6 per sheet, 3x2, about 640 px each, numbered) to save context; open a full-size photo only when a detail matters (mechanical labels, defects). Write a photo log in the scratchpad (photo-log.md), one line per photo: file name, what it shows (front, rear, side or street view; room and level; mechanical; amenity; defect close-up; crop), and what it shows that the report needs:
   - exterior: foundation, exterior walls, roof surface, windows, gutters and downspouts, and their condition
   - mechanicals: heating type and fuel (furnace or boiler, gas meter), cooling, water heater, electrical panel
   - interior: flooring, walls and trim, ceilings, kitchen and bath finishes and fixtures, appliances, fireplace, smoke and CO detectors, accessibility features (grab bars)
   - amenities: patio, deck, porch, pool, fence, sheds and other outbuildings
   - every defect or item of deferred maintenance
2. Assign each room photo a level, Finished or Unfinished, a room type and a room name, matched to the sketch or floor plan (e.g. on a split-level, the lower family room is Level 1 and the main living area is Level 2; basement and garage photos are Unfinished). Where a photo and the sketch disagree, use the sketch and flag it.
3. Compress copies (JPEG, long edge about 1600 px, each under 5 MB) into Scratch/<slug>/uploads/ so they can be attached by absolute path. Copy the floor plan, calculation pages, plat, zoning and flood maps there too.
4. Photos go straight onto each section's own exhibit or photo input as that section is filled in Step 3, each with its category and a short caption. There is no need to upload them to the Workfile panel first.
5. Defect close-ups and crops are used as defect exhibits on the page where that defect is entered.
6. List what the photos do not show that the report or the AMC needs (Class requires photos of the water heater, smoke and CO detectors, all kitchen appliances, front, rear and street). These go in the summary as photos to request.

Enter only what a photo clearly shows. Anything a photo leaves unclear is left blank and flagged, never guessed. The photo log stays in the scratchpad as the support for every photo-based entry.

## Step 3. Work through the report
Use the left navigation panel (open it with the Navigation icon at top left if it is collapsed; get each nav button's ref with find and click it, since clicking by text match can toggle the section open or closed instead of navigating). Go section by section, top to bottom, scrolling to the end of every page. Track what is left by the red bubbles in the left navigation; do not use Aivre's Validation panel.

The job is done when the red bubbles are gone for every section before Sales Comparison Approach, except where the information truly is not available. Never invent a value just to clear a bubble.

On each page:
- Enter every field by hand from the documents, the AMC emails and the photo log. See "Browser method" for the fast way to do it.
- Set dropdowns and chips directly with A.pick and A.chip, using the exact option the documents and these skills call for. Do not open a dropdown just to read it first. Never type free text into a dropdown. If A.pick returns NOT FOUND, it has entered nothing and returned the live options: choose from those. If A.chip returns NOT FOUND, see its options with A.chipOpen, press Escape (A.esc()), then A.chip again.
- If the appraiser has already entered something, keep it unless a document contradicts it, and note the conflict.
- Never write "Unknown", "Cannot Determine", "not observable" or similar placeholder text. Leave the field blank and flag it.
- Replace leftover placeholder text (e.g. "I will add this later") once the real data is entered.
- When report data changes, update any commentary that mentions it.

### Section by section

**Assignment Information**
- Assignment Reason (Purchase, Refinance, ...); Property Valuation Method (Traditional / Hybrid / Desktop / Exterior Appraisal) Traditional and "Was a Property Data Report used in lieu of an Inspection?" No (aivre-fill-hybrid changes both); Appraiser Fee; AMC Fee only when the state or client requires it; Government Agency Appraisal No unless FHA/VA/USDA.
- Borrower from the AMC email or order page, entered once. Sellers: the contract, or for a refinance leave blank (flag it). Current Owner of Public Record from the tax record or county property card. Each "Add name" control must be clicked by its find ref; then fill only the First/Last Name inputs inside that block (the next First Name inputs further down belong to the Appraiser and the PDC).
- Appraiser: the name, company and address in appraiser-profile.md. Credentials are pre-filled; leave them. Scope of Inspection: Physical/Physical with the inspection date for a traditional interior.
- Contacts: one card each. The lender as Client/Lender with its address from the order; the AMC as Non-Client/Appraisal Management Company with its address, state registration ID, state and expiration. The nightly task's report may already have a card with the AMC name as Client: change it to the lender and add the AMC with "Add another contact" > Add new.
- Assignment Information and Scope of Work Commentary: separate one-sentence paragraphs with a blank line between, in this order (aivre-fill-hybrid adds its paragraph after the first):
  - "The assignment is a <conventional/FHA/etc.> <purchase/refinance> transaction."
  - "The appraisal management company's Illinois registration number is <#> and it expires on <MM/DD/YYYY>."
  - "The compensation for this appraisal assignment is $<fee>." When the engagement letter prescribes fee disclosure wording (aivre-fill-hybrid has Class's hybrid wording), use that wording in place of this sentence.
  - Always last: the state disclaimer paragraph in appraiser-profile.md, word for word (leave it out when the profile says None).
- Assignment Information Exhibits usually already hold the license; keep it. If the AMC wants the E&O page and it is not in the workfile, flag it.

**Subject Property**
- County from the record (say so in commentary when the order differs). Neighborhood Name: the subdivision from the county record or MLS.
- Yes/No rows: Planned Unit Development, Property in a Project, Subject Site Owned in Common and Homeowner Responsible for Exterior Maintenance as the property-type skill says; Property on Native American Lands No; New Construction (Yes only for proposed, under construction, or completed and never occupied; No once occupied. A recently built home is still flagged when the engagement letter has a new-construction stop rule); Special Tax Assessments (record or MLS).
- Subject Property Photo (required, its own file input): the front view. Attachment Type from the documents. Dwelling and unit counts are read-only and follow the dwellings and units in the left nav (aivre-fill-2-4-unit adds units).
- Property Rights Appraised: Fee Simple, then "All Rights Included in Appraisal" Yes.
- Legal description: use the IMAGE whenever the workfile has one, never a transcription. Crop a copy (not the workfile original) down to only the legal description paragraph itself (the bold lot/unit/subdivision text) with a little margin; leave out the deed boilerplate above and the PIN, address and "subject to" lines below. View the crop before uploading. Set "Legal Description Image Indicator" to Yes, upload the cropped image (find the page's file input and attach the image with DOM.setFileInputFiles; do not click the upload box), and caption it "Legal Description". Do not type the legal description into the text field.
- Exception: Lake County and Will County orders have no photo, so enter the legal text from the public record (Will County: the property card's Legal Description) followed by the PIN, and leave the indicator No. If no public record is in the workfile, leave it red and flag it.
- Run "Create aerial". Check the pin against the plat, zoning and flood captures before saving. If the pin is on the wrong parcel, click Change next to the Subject, type the correct latitude and longitude, and click Set pin. Then drag the SUBJECT balloon up and to the side so it sits off the house, with its pointer leading to the subject; the house itself must be visible, not covered by the balloon. Saving can take 30+ seconds; wait for it rather than closing the modal. In new subdivisions the imagery may predate the house; note that in the summary.
- Latitude and longitude: the confirmed aerial pin's coordinates, entered to 6 decimals (more decimals leaves the field red; Aivre rounds to 5 after saving the aerial, so re-enter them). After the layout shifts, re-find the field before typing, or the coordinates land in the commentary box.
- Subject Property Commentary from the order, record, plat and a usable MLS listing only: county conflicts, PUD and association dues (state both amounts when sources differ), an age-restricted (55+) community.

**Site**
- Site size as the property-type skill says.
- Total Site Size and Number of Parcels, then Add Parcel for each: APN (the PIN) and APN Description as the property-type skill says.
- Zoning: Compliance (Legal / Legal Nonconforming / Illegal / No Zoning), then the Classification Code and Classification Code Description fields that appear (re-find fields after the layout shifts). Legal Nonconforming also requires an Impact (Adverse / Beneficial / Neutral) and the Zoning Compliance Description (246 characters).
- Non-Residential Use Exists No for a residential property.
- Property Access: Primary Access Public Street or Private Street (a "(PVT)" label on the county tax map or property card means private). Either one opens "Street Types and Surface Materials", which needs "Add Street": Street Used to Access Property (usually Local Road) and Street Surface Material (from the street photo). Private Street also adds Known Maintenance Agreement (No unless a document shows one; flag it, the engagement letter may require comment on private roads). Access to Property Typical for the Market Yes. Description of Property Access: one or two sentences, naming the source when sources conflict. Property Access Photo (required): a street view, caption "Street View"; set the caption after the upload because the upload clears it.
- Site Influence: Residential / Bordering / Neutral unless documented otherwise. Apparent Environmental Conditions: "None". Views: View Type, Is Primary View, Range of View, Impact (typically Residential, Yes, Full, Neutral; flag it). Hazard Zone: Zone X is entered as "No Hazard Zone Noted" (Add Item). Without a flood map in the workfile, use the zone from a prior appraisal of the subject if one is in the workfile and flag it; otherwise leave it and flag it.
- Property Restrictions: "Age" with impact Neutral and a short comment for an age-restricted or 55+ community; others only as documented. Easements (impact Neutral) and encroachments as documented.
- Site Characteristic "None". Broadband Internet Available Yes in served urban and suburban areas (flag it). Utilities: four tabs (Electricity, Gas, Sanitary Sewer, Water); click each tab and set its Utility Ownership (Public or Non Public) from the record or MLS.
- Site Defects No unless a document or photo says otherwise.
- Site Commentary: the site size source and any conflict, and the flood zone with FIRM panel and date.
- Site Exhibits: the plat map (category Property Boundaries, caption "Plat Map"), the survey, the zoning map and the flood map (Site Exhibit, captions "Zoning Map", "Flood Map"), Big Photo.

**Disaster Mitigation**: Mitigation Feature "None" unless documented.

**Energy Efficient and Green Features**: Known Renewable Energy Components, Known Building Certifications and Known Green Efficiency Ratings No unless documented (check the MLS for a HERS score on newer homes).

**Sketch**
- If the appraiser provides a finished sketch image (Sketch.png from TOTAL Sketch), use it as is. The appraiser handles sketches themselves; never draw one unless they ask.
- If the appraiser asks ChatGPT to draw the sketch, follow the property-type skill's drawing rules.
- In Aivre: Sketch or Floor Plan Provided Yes, Measurement Standard ANSI (aivre-fill-condo and aivre-fill-2-4-unit change it; "Other (Describe)" turns the field into a 50-character text box), clear any placeholder commentary, then Add Exhibit: upload the image (Image Category Sketch, caption "Sketch", Photo Size Big Photo). To revise it, use the exhibit's Replace photo file input rather than adding a second exhibit.
- Sketch Commentary: the measurement source and standard, and any discrepancy between the floor plan, the photos and other sources (builder/MLS or assessor GLA vs measured, room labels).
- Then make the report match the sketch: Unit Interior Finished Above Grade and Level 1 Finished Area = the rounded living area, below-grade fields 0 when there is no basement, room table matching the sketch rooms, Vehicle Storage garage area, and Subject Property Amenities patio/deck/porch areas.

**Dwelling**
- General: Dwelling Style (and Structure Design for a condo) as the property-type skill says, Front Door Elevation (from photos; flag it), Year Built and Year Built Estimated, Construction Method (multi-select chip, usually Site Built), Converted Area No. Year Built can come from any source that has it (public record, an MLS listing of any age, any other document in the workfile); flag it only when sources disagree.
- Dwelling Front Photo (required, its own file input, separate from the exhibits): the front view.
- Exterior Quality Rating and Exterior Condition Rating: from the photos and documents; list under Judgment calls.
- Exterior Features: one tab per feature, each with its own Condition Status (New or Like New / Typical Wear and Tear / Damaged and Functional / Damaged and Nonfunctional):
  - Exterior Walls and Trim: Wall Materials (multi-select, e.g. Vinyl and Brick).
  - Foundation: Foundation Materials, then "Add Foundation Type" (Basement, Crawl Space, Slab, ...).
  - Roof: Roof Observable, Roof Materials, Estimated Age.
  - Windows: Exterior Feature Description (e.g. "Vinyl").
  - Gutters and downspouts: add a feature with the "+" next to the tabs, Exterior Feature "Other (Describe)" = "Gutters and Downspouts", description e.g. "Aluminum".
- Noncontinuous Finished Area Exists No. Heating: "Add Heating System" with Heating System Type and Fuel. Cooling System Exists and Cooling System Type (Centralized / Individual). Core Heating System Below Grade. Other Mechanical System Type (Water Heater, Sump Pump, Radon Mitigation, ...) only when seen or documented; leave blank and flag what neither photos nor documents show.
- Dwelling Exterior Defects Exist No unless documented or seen in the photos. Short exterior commentary.
- Exterior exhibits (categories Dwelling Front, Dwelling Rear, Noncontinuous Area, Dwelling Exterior Exhibit): always the front ("Front View") and the rear ("Rear View"); side and street views when there are photos ("Side View"); mechanicals each as Dwelling Exterior Exhibit with a short caption (Water Heater, Furnace, Boiler, Electrical Panel, Air Conditioning Condenser).

**Unit Interior**
- Area: Finished Above Grade from the sketch, otherwise survey or assessor; the five other area fields (nonstandard, unfinished, below grade) 0 when there is none. Area Data Source is ALWAYS "Physical Measurement" chosen from its list (a chip field: its "+" Add option button) (aivre-fill-hybrid has the one exception). Never Plat of Survey, Assessor Record, MLS or an "Other (Describe)" entry. Level areas must equal the totals.
- Unit is an ADU No. Levels in Unit. Occupancy (Owner / Tenant / Vacant) from the order. Total Bedrooms, Full and Half Baths from the sketch; Total Bedrooms must equal the number of Bedroom rooms in the room table. Non-Residential Use and Live/Work Space: No for a residential unit.
- Level and Room Details: "Add Level" (Level in Unit, Grade Level Type, Finished Area, Unfinished Area), then one room tab per room with "Add Room Type". There is no room-name field: the room name goes in the room photo's caption (e.g. "Primary Bedroom", "Primary Bathroom"). Kitchen and bath tabs require a photo; attach one photo to every room that has one. An office is room type "Other (Describe)" with the description "Office". Foyers, halls, closets and the garage are not entered as rooms.
- Kitchen and Bathroom Details: one tab per kitchen and bath (created by the room table), each with Update Status (Fully / Partially / Not Updated) and Condition Status; then Overall Update Status for Bathrooms. A home built recently and never renovated is Not Updated, New or Like New. Fully or Partially Updated adds a required Time Frame (Less than 1 year / 1-5 years / 5-10 years / 10 or more years): only from a document (seller update list, permits, the appraiser's notes) or work still in progress in the photos (Less than 1 year); otherwise leave it blank and flag it.
- Interior Features: Flooring ("Add Flooring Type" per material, each with Condition Status; a flooring photo; Overall Update Status for Flooring) and Walls and Ceiling (Ceiling Heights and Ceiling Styles multi-selects, Condition Status, a photo). Take ceiling height from the MLS remarks when stated; otherwise judge from the photos and flag it.
- Accessibility Features (required): "None", or what the photos or documents show (e.g. Grab Bars) with a short comment.
- Unit Interior Defects Exist No unless documented or seen. Interior commentary follows the appraiser's condition statement (see Overall Quality and Condition). For a recent renovation, describe the scope and quality of the updating in a short general writeup; never "gut-rehab" unless the home was taken to the studs.
- Unit Interior Exhibits (category fixed to Unit Interior Exhibit): smoke and CO detectors, appliances not already on room photos, an interior water heater, and other useful views, each with a short caption.

**Deferred maintenance / defects**
- Every defect in the photo log is entered. Cost to cure from the MVS Cost Handbook in the workfile (Markdown copy), with current cost and local multipliers. If the handbook copy is not in the workfile, enter a reasonable estimate and flag both the estimate and the missing handbook. Each defect's Description states "This does not affect the safety, soundness, or structural integrity of the improvements." and ends with "Estimated cost to cure is $X." Cosmetic items: Affects Soundness No, Recommended Action None (Rocket and as-is rule). Do not wait for the appraiser to confirm costs; they review at the end.
- Water stains or possible leaks that call for a roof or attic inspection: Affects Soundness No, Recommended Action Inspection, and note in the condition commentary that the appraisal is subject to the inspection based on the extraordinary assumption that no repairs are required.
- Put each defect on its own feature's page: siding/trim/roof/windows/foundation under Dwelling exterior defects; walls, ceilings and flooring under Unit Interior defects. Deck and patio defects go ONLY on the Subject Property Amenities > Outdoor Living page under that amenity, never on the Dwelling page.
- Use the close-up and cropped detail photos as defect exhibits.
- Also deliver the list in the appraiser's format ("List of Deferred Maintenance Observed-", numbered "<item>. Estimated cost to cure is $X.") in chat and as a Markdown file, and put the total in the Overall Quality and Condition reconciliation. With no defects, say so in the summary.

**Functional Obsolescence**: Functional Issues "None" unless documented, with Functional Obsolescence Commentary "No irregularities are observed."

**Vehicle Storage**: "Add Vehicle Storage" for each type. Garage: Dedicated Parking Spaces, Attachment Type (Attached / Built-in / Detached), Vehicle Storage Area (required), and a garage photo. Driveway: "10 or More Parking Spaces", spaces, Driveway Surface Material. Garage area from the sketch or the survey; otherwise leave it red and flag it. Garage commentary must not cite an old MLS.

**Subject Property Amenities**: set Property Amenities Exist first. Categories: Outdoor Accessories, Outdoor Living (Balcony, Deck, Gazebo, Patio, Porch, Portico; Material; Amenity Area is required; a photo with caption), Water Features, Whole Home, Miscellaneous. Amenities from the photo log; areas from the sketch. When the sketch draws a porch or patio without an area, calculate it from the dimension labels and flag it.

**Overall Quality and Condition**
- Follow the appraiser's condition instruction. Example: property being renovated, C5 for the missing flooring/trim being installed, C4 otherwise. If the AMC has a C5 rule (Solidifi), add the cost-to-cure note or flag that the amount is needed.
- With no instruction from the appraiser, set Overall Quality and Overall Condition from the photo log and the documents and list them under Judgment calls. C1 only for new construction never occupied; an occupied new home is C2.
- Reconciliation of Overall Quality and Condition: short factual commentary that always includes "Nothing affects the safety, soundness, or structural integrity of the improvements." (even though there is a checkbox) and, always, this exact paragraph: "Smoke detectors, carbon monoxide detectors and other life safety devices were not tested because testing them is outside the scope of this appraisal. The appraiser makes no representation as to their operation, age, condition, or remaining lifespan, and the owner and occupants should have them checked by a qualified professional." Never disclaim the detectors' presence.

**Highest and Best Use**: Yes on all four tests and Yes for "highest and best use is the present use" when the improvement is a conforming residential use in a residential zone, with a one or two sentence summary in the commentary (some AMCs require it).

**Market**
- The appraiser's market data usually comes as a connectMLS TSV export plus a "Relevant Sales and Listings (big photo).png" image of the same list. Compute from the TSV: actives = NEW + ACTV; pending = A/I + HS72 (and other contingent statuses); closed = CLSD in the lookback; leave out TEMP/off-market. Fill active count, median days on market of actives (MT), lowest/median/highest list price, pending count, sales count and lowest/median/highest sale price. Add the PNG as a Market Exhibit (Market Analysis Exhibit, caption "Relevant Sales and Listings", Photo Size Big Photo). Put the search criteria the appraiser states in the Search Criteria Description.
- If the appraiser gives only a list-of-sales image, compute the same counts and price ranges from it. Treat items they label (e.g. "A/I is pending") as they say.
- With no TSV or list in the workfile, fill only what the graphs support (below) and leave Market Area Boundary, Search Criteria and the listing and sale counts red, flagged.
- Market charts: crop each to the chart area only, from the Print/Share row down through the filter footnote, keeping the right-hand value panel and leaving out the filter panel above. When the workfile has CropGuide.png, locate it inside one chart screenshot by template matching and apply that box to all of them (a 1653x1464 chart crops to (0,328,1638,1385); a 1702x1464 chart crops to (19,297,1664,1386)). Without a guide, a 1484x735 Infosparks capture crops to (0,250,1468,735). View a crop before uploading.
- Add each chart with "Add Market Exhibit": category Price Trend (sale price and price per square foot graphs), Median Days on Market (median market time), or Market Analysis Exhibit (the rest); Photo Size Graph; short captions matching the chart (Median Sale Price, Average Sale Price, Median Price Per Square Foot, Average Price Per Square Foot, Median Market Time, Average Market Time, Median Percent of Original Price, Average Percent of Original Price).
- Price Trend Source from appraiser-profile.md (default "MRED Chicago Area MLS"). Price Trend Analysis Commentary covers price trends only (the 12-month values and changes from the graphs' right panels). Market Commentary covers supply, demand, marketing time and the search results. Demand/Supply and Marketing Time follow from median market time and percent of original list price; Distressed Market Competition No unless the data shows otherwise (flag it when there is no search data). Never use "Simply put".

**Project Information**: condo only, in aivre-fill-condo.

**Subject Listing Information**
- Every listing of the subject within the past year must be included. Check the subject's connectMLS listing history, not just one listing sheet: agents and builders take listings down and relist to reset days on market (a gap between a listing's days on market and its cumulative days is a sign), and PRIV-ACTV means the private network without full exposure.
- For each listing: "Add Listing" with Listing Status (Active / Off Market / Pending), Listing Type (MLS, FSBO, ...), Listing ID, start and end dates, Days on Market, starting and final list price. Then Total Days on Market and a short Analysis of Subject Property Listing History (dates, prices, contract and close, current status). If an earlier listing is indicated but not in the workfile, flag it.
- With no listing in the past year: "No" with data source MLS. Never use "Other (Appraiser research)".
- A subject currently listed on a refinance is usually an AMC stop-and-contact item; put it first in the summary.

**Sales Contract**
- Only from a purchase agreement the appraiser attached or that is in the workfile. With no contract (e.g. a refinance), set "Is there a sales contract?" to No.
- If the contract PDF is large, compress a copy (rasterize at about 130 dpi, JPEG quality about 50) and check the signature page is still legible; save it to the workfile as "Purchase Contract.pdf". Aivre's upload limit is 5 MB.
- NEVER add the purchase agreement as an exhibit (Sales Contract Exhibits stays empty).
- Fields: Is there a contract Yes; analyzed Yes; arm's length Yes unless the documents show otherwise; Contract Price from the contract (never an MLS list price); Contract Date = date of acceptance / full execution; Transfer Terms "Typically Motivated" for an ordinary sale (other options: Court Ordered Non-Foreclosure, Estate, Foreclosure, Land, Pre-Subdivision, Relocation, REO, Related Parties, Short Sale, Other); Known Sales Concessions No when the seller credit is N/A.
- Personal Property Conveyed: Yes when the contract lists appliances or other personal property (e.g. refrigerator, freezer, range, dishwasher, microwave, washer, dryer, window treatments), and say in the commentary they are not included in the appraised value.
- Use the appraiser's signing dates when they give them. Read the whole package for price changes, counteroffers, escalation forms and concessions. Check sellers against the owners of record.
- Commentary pattern (short, no em dashes, no speculation):
  "The sales contract was analyzed. It was signed by the buyer on MM/DD/YYYY and by the seller on MM/DD/YYYY. [Price change sentence if any, e.g. The purchase price was changed from $X to $Y and initialed by the parties.] There are no seller concessions and no financial assistance paid on behalf of the borrower. The sellers are the owners of public record. The contract includes the <items> at no added value. These items are personal property and are not included in the appraised value. [e.g. An intent to escalate clause was also provided but is not signed by the seller.]"
  Describe a side form accurately (who signed it, binding or not); do not say the contract "includes" a form the form itself says is not part of the contract, and do not imply a link between documents (e.g. escalation caused the price change) that no document states.

**Prior Sale and Transfer History**
- The subject's transfers within three years, from the public record or deed. A home built within the last three years has at least the builder's sale; find it.
- Prior Sales or Transfers Yes, then "Add Transfer": Ownership Transfer Type (Sale / Deed Transfer Only), Prior Sale Type (Typically Motivated for an ordinary or builder sale), Date and Amount from the record, Data Source Assessor Record (or Deed). There is no document-number field: put the document number in the analysis. When the MLS closing date differs from the recorded date, give both in the analysis.
- If the last transfer is older than three years, answer No and mention the deed in the analysis.
- Leave the comparable part to the aivre-sales-comparison skill.

**Stop at Sales Comparison Approach.** Skip it and continue with Reconciliation and Footer.

**Reconciliation**
- These defaults do not apply to 2-4 unit reports: the appraiser reconciles those themselves and the whole section stays blank (aivre-fill-2-4-unit).
- Income Approach Reason for Exclusion: Not Necessary for Credible Results and Insufficient Data. Cost Approach Reason for Exclusion: Not Necessary for Credible Results and Difficulty Estimating Depreciation. Market Value Condition As Is (unless the assignment is subject to repair, completion or inspection). Additional Client Requested Conditions No.
- Reconciliation of Market Value: "The cost and income approaches are not relevant. There is good recent sale data. The sales approach is given all of the weight."
- Leave Sales Comparison Indicated Value, Opinion of Market Value, Reasonable Exposure Time and Effective Date of Appraisal blank for the appraiser.

**Footer**: Client Reference ID = the loan number or client number, and AMC Reference ID = the AMC order number (e.g. Class "1234567.1"), when available. Leave the Appraisal Version Number and Appraiser Reference ID as they are.

**Certifications and Scope of Work**: leave it for the appraiser. Its red bubbles (Appraiser Subject Property Inspection, signature items) belong to signing and are expected.

## Rules
- Only enter what a document, the AMC emails or the photo log supports. Never guess.
- No Aivre AI features of any kind (Autofill, Commentary Fill Assist, AI suggestions, "Get AI help" buttons, Import from Sketch, Autofill from Photo, Ava AI Chat). Write all commentary directly.
- MLS listings of the subject: a listing active within the past year is a relevant listing. Enter it in Subject Listing Information, and its listing, contract and sale data (dates, prices, days on market, builder, subdivision, association dues) may be used and cited. Physical characteristics still come from the inspection, the sketch and the record; note material differences (e.g. builder GLA vs measured) in commentary.
- An older MLS listing (more than a year old) is supplemental. It may stay in the workfile but must not appear in the report or be referenced. Never enter values from it: commentary, area data source, Property in a Project/attachment type, fees, garage count, sales contract data, patio size. Year built is the exception: it can come from any source, an older listing included.
- Any commentary must be short, factual and pasteable, with no em dashes.
- Do not sign, submit, deliver, or change the report's status.

### Browser method
Setting fields one at a time with find, click, type and screenshot is slow. Inject this helper once per page load with CDP Runtime.evaluate (re-inject after any navigation or reload; see chatgpt-runtime.md). Values set through it autosave like typed input.

```js
window.A = {
 sleep: ms=>new Promise(r=>setTimeout(r,ms)),
 labs(t){ return [...document.querySelectorAll('label')].filter(l=>l.innerText.replace(/\s+/g,' ').replace(/\*/g,'').trim().startsWith(t)); },
 vis(t){ return A.labs(t).filter(l=>l.offsetParent); },
 visIdx(t){ return A.labs(t).indexOf(A.vis(t)[0]); },
 lastIdx(t){ return A.labs(t).indexOf(A.vis(t).pop()); },
 idxAfter(ref,t){ const r=A.vis(ref)[0]; const all=A.labs(t); return all.indexOf(all.find(l=>l.offsetParent && (r.compareDocumentPosition(l)&4))); },
 after(l,sel){ return [...document.querySelectorAll(sel)].find(e=>l.compareDocumentPosition(e)&4); },
 inp(t,i=0){ const l=A.labs(t)[i]; return l && A.after(l,'input:not([type=checkbox]):not([type=radio]):not([type=file]),textarea'); },
 btn(t,i=0){ const l=A.labs(t)[i]; return l && A.after(l,'button[aria-haspopup]'); },
 async setEl(el,v){ el.scrollIntoView({block:'center'}); el.focus(); const p=el.tagName==='TEXTAREA'?HTMLTextAreaElement.prototype:HTMLInputElement.prototype; Object.getOwnPropertyDescriptor(p,'value').set.call(el,v); el.dispatchEvent(new Event('input',{bubbles:true})); el.dispatchEvent(new Event('change',{bubbles:true})); await A.sleep(150); el.blur(); el.dispatchEvent(new FocusEvent('focusout',{bubbles:true})); await A.sleep(400); return el.value; },
 async set(t,v,i=0){ const el=A.inp(t,i); return el ? t+' = '+await A.setEl(el,v) : 'NO FIELD '+t; },
 async open(t,i=0){ const b=A.btn(t,i); b.scrollIntoView({block:'center'}); ['pointerdown','mousedown','pointerup','mouseup','click'].forEach(x=>b.dispatchEvent(new PointerEvent(x,{bubbles:true}))); await A.sleep(500); return [...document.querySelectorAll('[role=option]')].map(o=>o.innerText.trim()); },
 esc(){ document.dispatchEvent(new KeyboardEvent('keydown',{key:'Escape',bubbles:true})); },
 async pick(t,opt,i=0){ const os=await A.open(t,i); const o=[...document.querySelectorAll('[role=option]')].find(e=>e.innerText.trim()===opt); if(!o){ A.esc(); return 'NOT FOUND '+t+': '+os.join(' | '); } o.click(); await A.sleep(600); return t+' = '+(A.btn(t,i)||{innerText:'?'}).innerText.trim(); },
 chipBtn(t){ const l=A.vis(t)[0]; return [...document.querySelectorAll('button[aria-label="Add option"]')].find(b=>b.offsetParent && (l.compareDocumentPosition(b)&4)); },
 async chipOpen(t){ const b=A.chipBtn(t); b.scrollIntoView({block:'center'}); ['pointerdown','mousedown','pointerup','mouseup','click'].forEach(x=>b.dispatchEvent(new PointerEvent(x,{bubbles:true}))); await A.sleep(600); return [...document.querySelectorAll('[role=option],[role=menuitem],[role=menuitemcheckbox]')].filter(o=>o.offsetParent).map(o=>o.innerText.trim()); },
 async chip(t,opts){ const out=[]; for(const opt of opts){ await A.chipOpen(t); const o=[...document.querySelectorAll('[role=option],[role=menuitem],[role=menuitemcheckbox]')].find(e=>e.offsetParent && e.innerText.trim()===opt); if(o){ o.click(); await A.sleep(700); out.push(opt);} else { A.esc(); out.push('NOT FOUND '+opt); } } A.esc(); await A.sleep(300); return t+': '+out.join(','); },
 textEl(t){ const m=[...document.body.querySelectorAll('*')].filter(e=>e.offsetParent && [...e.childNodes].some(n=>n.nodeType===3 && n.textContent.trim().replace(/\*$/,'').trim()===t)); return m[m.length-1]; },
 async yn(t,c){ const el=A.textEl(t); if(!el) return 'NO ROW '+t; const r=[...document.querySelectorAll('[role=radio]')].find(x=>(el.compareDocumentPosition(x)&4) && x.getAttribute('aria-label')===c+' option'); r.scrollIntoView({block:'center'}); if(r.getAttribute('aria-checked')!=='true'){ r.click(); await A.sleep(700); } return t+' = '+c; },
 async clickBtn(t){ const b=[...document.querySelectorAll('button')].find(e=>e.offsetParent && (e.innerText.trim()===t || e.getAttribute('aria-label')===t)); if(!b) return 'no visible '+t; b.scrollIntoView({block:'center'}); b.click(); await A.sleep(1200); return 'clicked '+t; },
 async nav(t){ const b=[...document.querySelectorAll('button')].find(e=>e.offsetParent && e.innerText.trim()===t); if(!b) return 'no nav '+t; b.click(); await A.sleep(1500); return 'nav '+t; },
 async tab(t){ const b=[...document.querySelectorAll('button,[role=tab]')].find(e=>e.offsetParent && e.innerText.split('\n')[0].trim()===t && !e.getAttribute('aria-haspopup')); if(!b) return 'no tab '+t; b.click(); await A.sleep(800); return 'tab '+t; },
 async addExh(head){ const h=A.textEl(head); const b=[...document.querySelectorAll('button')].find(b=>b.offsetParent && (h.compareDocumentPosition(b)&4) && !b.innerText.trim() && !b.getAttribute('aria-label')); if(!b) return 'no plus'; b.click(); await A.sleep(900); return 'added'; },
 markUpload(lab){ document.querySelectorAll('[data-cc-target]').forEach(e=>{e.removeAttribute('data-cc-target'); e.removeAttribute('aria-label');}); const l=[...document.querySelectorAll('label')].find(l=>l.offsetParent && l.innerText.trim().startsWith(lab)); if(!l) return 'no label'; let p=l.parentElement, f=null; for(let j=0;j<6 && p && !f;j++){ f=p.querySelector('input[type=file]'); p=p.parentElement; } if(!f) return 'no file input'; f.setAttribute('aria-label','CCUPLOADTARGET'); f.setAttribute('data-cc-target','1'); return 'marked'; },
 async cap(text){ const c=[...document.querySelectorAll('input,textarea')].filter(i=>i.offsetParent && /caption/i.test(i.placeholder||'') && !i.value); return c.length ? 'cap '+await A.setEl(c[0],text) : 'no empty caption'; },
 async stateSel(code){ await A.sleep(400); const lb=document.querySelector('[role=listbox]'); if(!lb) return 'no listbox'; const si=lb.querySelector('input')||lb.parentElement.querySelector('input'); si.focus(); Object.getOwnPropertyDescriptor(HTMLInputElement.prototype,'value').set.call(si,code); si.dispatchEvent(new Event('input',{bubbles:true})); await A.sleep(500); const o=[...lb.querySelectorAll('button,[role=option]')].find(x=>x.innerText.trim()===code); if(!o) return 'no option'; o.click(); await A.sleep(600); return 'ok '+code; },
 bubbles(){ const c=[...document.querySelectorAll('div,nav,aside')].filter(e=>e.innerText && e.innerText.includes('Assignment Information') && e.innerText.includes('Footer')).sort((a,b)=>a.innerText.length-b.innerText.length)[0]; const L=c.innerText.split('\n').map(s=>s.trim()).filter(Boolean); const o=[]; for(let i=0;i<L.length-1;i++) if(/^\d+$/.test(L[i+1]) && !/^\d+$/.test(L[i])) o.push(L[i]+':'+L[i+1]); return o; },
 redf(){ return [...document.querySelectorAll('button,input,textarea,div')].filter(e=>e.offsetParent && /border-red-400/.test(String(e.className)) && e.getBoundingClientRect().x>210).map(e=>{ let p=e, lab=''; for(let k=0;k<5 && p && !lab;k++){ p=p.parentElement; const l=p&&p.querySelector('label'); if(l) lab=l.innerText.trim().slice(0,40);} return lab.replace(/\n/g,' ')+' ['+(e.innerText||e.placeholder||'').slice(0,30).replace(/\n/g,' ')+']'; }); },
 vlabels(){ return [...document.querySelectorAll('label')].filter(l=>l.offsetParent).map(l=>l.innerText.replace(/\s+/g,' ').trim().slice(0,60)); }
};
```

- Use: `await A.open('Label')` lists a dropdown's options (only when a pick missed or the list is needed; A.pick opens the dropdown itself); `A.pick` for dropdowns, `A.chipOpen`/`A.chip` for chip fields (Construction Method, Wall Materials, Area Data Source, Ceiling Heights...), `A.set` for text and textareas, `A.yn` for Yes/No rows, `A.tab` for a sub-form tab, `A.nav` for a visible nav button, `A.bubbles()` for the remaining red bubbles, `A.redf()` for the fields still red on the page, `A.vlabels()` to see the fields now visible. Several fields can be set in one javascript call; return the results and check them.
- The JS tool's output is blocked ("Cookie/query string data") when the returned text contains ";" or "&" (e.g. a URL query or a value with a semicolon). The action still runs; return lengths or strip those characters.
- "Get AI help for <field>" (a sparkle button) sits right after many labels. Never click "the first button after a label": it opens Ava AI Chat with a pre-filled prompt (close the panel without sending if it happens). Chip "+" buttons have aria-label "Add option"; `A.chipBtn` targets those.
- Labels repeat (Street Address, State, Company Name, Comment, Impact, Condition Status, Photo Caption). Pass the index of the visible one with `A.visIdx(label)` (`A.lastIdx` for the last visible one), or `A.idxAfter(refLabel, label)` for the first visible one after another field. Tabbed sub-forms (utilities, exterior features, rooms, kitchen/bath details, flooring, amenities, exhibits) keep every tab's panel in the page with only the active one visible, and repeat their "Add ..." buttons per panel. Click the tab (`A.tab` or a find ref), then target the visible field; index 0 may be a hidden tab's copy and silently do nothing.
- The first item in a sub-form is added with its "Add ..." button; later items (second exhibit, room, flooring type, amenity, exterior feature) with the unlabeled "+" at the left of the tab row (`A.addExh(heading)`).
- Yes/No rows are DIV[role=radio] elements labeled "Yes option"/"No option", not checkboxes; `A.yn` handles them. Row text like "Site Commentary" also appears in the left nav; `A.textEl` takes the last match, which is the page content.
- Some single-line fields are textareas (PDC Company Name, Price Trend Source); `A.inp` includes textareas. Never take "the first input after a heading" without checking which field it is.
- A hidden Ava AI Chat textarea ("Ask about your order...") sits early in the page. Never type into it.
- State dropdowns are a [role=listbox] with a search input over a virtualized list (only about 20 states are rendered). `await A.open('State', i); await A.stateSel('IL')` works; do one State at a time and read it back (clicking several State refs in a row once set the wrong state). Typing a street address opens an address autocomplete listbox; press Escape before the State step.
- "Add name" controls (borrower, seller, owner) work reliably only by find ref. Nav buttons: `A.nav(name)` works for a visible nav button; use a find ref for one that is collapsed or scrolled away.
- File uploads: hidden tab panels' file inputs are in the accessibility tree too, so find can return the wrong room's or exhibit's input. Mark the right one first: `A.markUpload('<label text>')` (e.g. 'Kitchen Photos', 'Exhibit 2 Image', 'Defect Photo'), then attach the file to input[data-cc-target="1"] with DOM.setFileInputFiles. Each new exhibit, room or item creates a new input, so mark and find again after each add. Set the caption after the upload (`A.cap(text)` fills the first empty "Caption for image" box), because uploading can clear a caption typed first. A wrong photo is removed from its kebab menu (Edit Photo / Replace / Delete).
- After a section, check A.bubbles(); after risky edits, reload the report and read the values back.
- Aivre's layout shifts after a field loses focus (character counters, new fields). Do not type or click by remembered coordinates. Blur by clicking a section heading, not an exhibit tab (that switches exhibits).
- Tabbing between fields does not work well; commit entries with the mouse or the helper.
- Aivre opens a saved-comments (Favorites) dropdown on text fields. It closes on blur. Never click the save icon next to a field label; that saves the text as a favorite. The green "Save" badge on a contact card saves it to saved contacts; it is not needed.
- A session timeout appears after about 15 minutes idle; click "Resume Editing".
- Dropdown search boxes: clear with repeated Backspace, not ctrl+a.
- Delete confirmations are in-app modals (click Remove), not browser dialogs. Many exhibits can be removed with a short javascript loop that clicks each "Remove item" button and then the modal's Remove button.
- Uploading a PDF to an exhibit opens a "Select PDF Pages to Convert" dialog that converts each page to its own exhibit, which is slow, especially when the tab is in the background.
- A batch of many browser actions can time out; do a few at a time and confirm each saved.
- Put compressed photos, crops and generated sketches in Scratch/<slug>/uploads/ and attach them by absolute path. Exhibit uploads are limited to 5 MB per file.
- If a photo or exhibit input is not where this skill describes, find it on the page and say where it was in the summary so the skill can be updated.
- On the dashboard list, open a report by clicking its address text, never the row's empty space or the Due column (that opens a due-date picker).
- Leave the report tab open when you finish edits the appraiser asked for in this conversation (a sketch swap, a section fill, a fix), so they can review it. Close the Chrome tabs you opened only in autonomous or unattended runs with no one watching, such as the nightly data fetch or other scheduled tasks.

## When done
Send the appraiser a short summary. Start with one line naming the modules used (e.g. single-family + hybrid) and what showed the type, then:
1. **Check first:** engagement-letter stop-and-contact items that apply (new construction, subject listed on a refinance, private road, GLA vs record), county or location problems, and anything else that could change the assignment.
2. **Filled:** by section, what you entered and its source (document or photo log).
3. **Conflicts:** where an order, an existing entry, the PDR, the MLS or the photos disagreed with a document (county, owner, rooms, road, site size, GLA, HOA dues) and what you used.
4. **Judgment calls to confirm:** Q/C ratings and condition statuses, room levels, features read only from photos, calculated areas, and anything entered as a default (None/No/Neutral, Occupancy, structure type, and the defaults the modules list).
5. **Still red:** each remaining bubble and why the information was not available.
6. **To request:** missing photos and missing documents (public record, listing history, E&O).